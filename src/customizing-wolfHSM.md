# Customizing wolfHSM

wolfHSM provides multiple points of customization 

### Library Configuration

The wolfHSM library has a number of build options that can be turned on or off though compile time definitions. The library expects these configuration macros to be defined in a configuration header named `wh_config.h`. This file should be defined by applications using wolfHSM and located in a directory in the compilers include path.

An example `wh_config.h` is distributed with every wolfHSM port providing a known good configuration.

For a full list of wolfHSM configuration settings that can be defined in `wh_config.h`, refer to the [API documentation]().

### DMA Callbacks

### Custom Callbacks

The custom callback feature in wolfHSM allows developers to extend the functionality of the library by registering custom callback functions on the server. These callbacks can then be invoked by clients to perform specific operations that are not covered by the default HSM capabilities such as enabling or disabling peripheral hardware, implementing custom monitoring or authentication routines, staging secure boot for an additional core, etc.

### Server side

The server can register custom callback functions that define specific operations. These functions must be of type `whServerCustomCb`.

```
/* wh_server.h */

/* Type definition for a custom server callback  */
typedef int (*whServerCustomCb)(
    whServerContext* server,   /* points to dispatching server ctx */
    const whMessageCustomCb_Request* req, /* request from client to callback */
    whMessageCustomCb_Response*      resp /* response from callback to client */
);

```

Custom server callback functions are associated with unique identifiers (IDs), which correspond to indices into the server's custom callback dispatch table. The client will refer to the callback by it's ID when it requests invocation.

The custom callback has access to data passed from the client by value or by reference (useful in a shared memory system) through the `whMessageCustomCb_Request` argument passed into the callback function. The callback can act on the input data and produce output data that can be passed back to the client through th e`whMessageCustomCb_Response` argument. The custom callback does not need to handle sending or receiving any of the input / output client data, as this is handled externally by wolfHSM. The response structure also contains fields for an error code and return code to propagate back to the client. The error code should be populated by the callback, and the return code will be set the the return value from the custom callback.

### Client Side

Clients can send requests to the server to invoke these custom callbacks. The API provides a request and response function similar to the other functions in the client API. The client should declare an instance of a custom request structure, populate it with its custom data, and then send it to the server using `wh_Client_CustomCbRequest()`. The server response can then be polled using `wh_Client_CustomCbResponse()`, and the response data will populate the output `whMessageCustomCb_Response()` on successful receipt.

The client can also check the registration status of a given callback ID using the `wh_Client_CustomCheckRegistered()` family of functions. This function queries the server for whether a given callback ID is registered in its internal callback table. The server responds with a true or false indicating the registration status.

### Custom Messaging

The client is able to pass data in and receive data from the custom callbacks through the custom request and response message data structures.
These custom request and response messages are structured to include a unique ID, a type indicator, and a data payload.  The ID corresponds to the index in the server's callback table. The type field indicating to the custom callback how the data payload should be interpreted.
The data payload is a fixed size data buffer that the client can use in any way it wishes. The response structure contains additional error code values described above.

```c
/* request message to the custom server callback */
typedef struct {
    uint32_t               id;   /* indentifier of registered callback  */
    uint32_t               type; /* whMessageCustomCb_Type */
    whMessageCustomCb_Data data;
} whMessageCustomCb_Request;

/* response message from the custom server callback */
typedef struct {
    uint32_t id;   /* indentifier of registered callback  */
    uint32_t type; /* whMessageCustomCb_Type */
    int32_t  rc;   /* Return code from custom callback. Invalid if err != 0 */
    int32_t  err;  /* wolfHSM-specific error. If err != 0, rc is invalid */
    whMessageCustomCb_Data data;
} whMessageCustomCb_Response;
```

### Defining Custom Data Types

Custom data types can be defined using the `whMessageCustomCb_Data` union, which provides several helpful predefined structures for common data types (e.g., dma32, dma64) and a raw data buffer (buffer) for user-defined schemas. Clients can indicate to the server callback how it should interpret the data in the union through the `type` field in the request. wolfHSM reserves the first few type indices for internal use, with the remainder of the type values available for custom client types.


### Custom Callback Example

In this example, a custom callback is implemented that is able to process three types of client requests, one using the built-in DMA-style addressing type, and two that use custom user defined types.

First, common messages shared between the client and server should be defined:

```c
/* my_custom_cb.h */

#include "wolfhsm/wh_message_customcb.h"

#define MY_CUSTOM_CB_ID 0

enum {
	MY_TYPE_A = WH_MESSAGE_CUSTOM_CB_TYPE_USER_DEFINED_START,
	MY_TYPE_B, 
} myUserDefinedTypes;

typedef struct {
	int foo;
	int bar;
} myCustomCbDataA;

typedef struct {
	int noo;
	int baz;
} myCustomCbDataB;
```

On the server side, the callback must be defined and then registered with the server context before processing requests. Note that the callback can be registered at any time, not necessarily before processing the first request.

```c
#include "wolfhsm/wh_server.h"
#include "my_custom_cb.h"

int doWorkOnClientAddr(uint8_t* addr, uint32_t size) {
	/* do work */
}

int doWorkWithTypeA(myCustomTypeA* typeA) {
	/* do work */
}

int doWorkWithTypeB(myCustomTypeB* typeB) {
	/* do work */
}

static int customServerCb(whServerContext*                 server,
                          const whMessageCustomCb_Request* req,
                          whMessageCustomCb_Response*      resp)
{
	int rc;

	resp->err = WH_ERROR_OK;

	/* detect and handle DMA request */
    if (req->type == WH_MESSAGE_CUSTOM_CB_TYPE_DMA32) {
		uint8_t* clientPtr = (uint8_t*)((uintptr_t)req->data.dma32.client_addr);
		size_t clientSz = req->data.dma32.client_sz;
		
		if (clientPtr == NULL) {
			resp->err = WH_ERROR_BADARGS;
		}
		else {
			rc = doWorkOnClientAddr(clientPtr, clientSz);
		}	
    }
	else if (req->type == MY_TYPE_A) {
		myCustomCbDataA *data = (myCustomCbDataA*)((uintptr_t)req->data.data);
		rc = doWorkWithTypeA(data);
		/* optionally set error code of your choice */
		if (/* error condition */) {
			resp->err = WH_ERROR_ABORTED;
		}
	}
	else if (req->type == MY_TYPE_B) {
		myCustomCbDataB *data = (myCustomCbDataB)((uintptr_t)req->data.data);
		rc = doWorkWithTypeB(data);
		/* optionally set error code of your choice */
		if (/* error condition */) {
			resp->err = WH_ERROR_ABORTED;
		}
	}

    return rc;
}


int main(void) {
	
	whServerContext serverCtx;

	whServerConfig serverCfg = {
		/* your server configuration */
	};

	wh_Server_Init(&serverCtx, &serverCfg);

	wh_Server_RegisterCustomCb(&serverCtx, MY_CUSTOM_CB_ID, customServerCb));

	/* process server requests (simplified) */ 
	while (1) {
		wh_Server_HandleRequestMessage(&serverCtx);
	}

}
```

Now the client is able to check the registration of the custom callback, as well as invoke it remotely:

```c
#include "wh_client.h"
#include "my_custom_cb.h"

whClientContext clientCtx;
whClientConfig  clientCfg = {
	/* your client configuration */
};

whClient_Init(&clientCtx, &clientCfg);

bool isRegistered = wh_Client_CustomCheckRegistered(&client, MY_CUSTOM_CB_ID);

if (isRegistered) {
	uint8_t buffer[LARGE_SIZE] = {/* data*/};
	myCustomCbDataA typeA = {/* data */};
	myCustomCbDataB typeB = {/* data */};

	whMessageCustomCb_Request req = {0};
	whMessageCustomCb_Request resp = {0};

	/* send custom request with built-in DMA type */	
	req.id = MY_CUSTOM_CB_ID;
    req.type                   = WH_MESSAGE_CUSTOM_CB_TYPE_DMA32;
    req.data.dma32.client_addr = (uint32_t)((uintptr_t)&data);
    req.data.dma32.client_sz   = sizeof(data);
    wh_Client_CustomCbRequest(clientCtx, &req);
    wh_Client_CustomCbResponse(clientCtx, &resp);
	/* do stuff with response */

	/* send custom request with a user defined type */
	memset(req, 0, sizeof(req));
	req.id = MY_CUSTOM_CB_ID;
    req.type = MY_TYPE_A;
	memcpy(&req.data.data, typeA, sizeof(typeA));
    wh_Client_CustomCbRequest(clientCtx, &req);
    wh_Client_CustomCbResponse(clientCtx, &resp);
	/* do stuff with response */

	/* send custom request with a different user defined type */
	memset(req, 0, sizeof(req));
	req.id = MY_CUSTOM_CB_ID;
    req.type = MY_TYPE_B;
	memcpy(&req.data.data, typeA, sizeof(typeB));
    wh_Client_CustomCbRequest(clientCtx, &req);
    wh_Client_CustomCbResponse(clientCtx, &resp);
	/* do stuff with response */
}

```

