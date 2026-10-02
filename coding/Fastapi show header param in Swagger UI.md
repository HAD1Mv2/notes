## Show header params in Swagger UI

When you use `request.headers.get('header-key')`, FastAPI reads the header directly from the raw HTTP request object behind the scenes. Because it happens dynamically inside your function, FastAPI's automatic documentation generator has no idea that your endpoint expects a header, so it shows "No parameters" in Swagger UI.

To fix this and make the input box appear in Swagger UI, you have two options:

### Option 1: The Modern FastAPI Way (Recommended)

The cleanest fix is to remove `request.headers.get` and instead declare the header explicitly in your function arguments using `Header()`. FastAPI will automatically read the header for you and generate the text box in Swagger UI.

``` python
from typing import Annotated
from fastapi import FastAPI, Header
app = FastAPI()

@app.get("/your-endpoint")
# FastAPI automatically handles hyphens/underscores, 
# so 'header_key' looks for 'header-key' in the headers
async def my_endpoint(header_key: Annotated[str | None, Header()] = None):
    # You can use 'header_key' directly now! No need for request.headers.get()
    return {"status": "success", "received_key": header_key}
```

### Option 2: Keep using request.headers.get but force Swagger UI to show it

If you absolutely must keep reading directly from the request object (for example, if you are migrating legacy code), you can leave your code as-is but inject a fake parameter parameter into the OpenAPI documentation using the openapi_extra argument in your route decorator.

``` python
from fastapi import FastAPI, Request
app = FastAPI()

@app.get(
    "/your-endpoint",
    # This manually forces Swagger UI to show the 'header-key' text box
    openapi_extra={
        "parameters": [
            {
                "name": "header-key",
                "in": "header",
                "required": False, # Set to True if it's mandatory
                "schema": {"type": "string"},
                "description": "Your custom application key"
            }
        ]
    }
)
async def my_endpoint(request: Request):
    # Your existing code stays exactly the same
    header_key = request.headers.get('header-key')
    return {"status": "success", "received_key": header_key}
```

Both options will immediately bring back the parameter input box under the "Try it out" button in Swagger UI! Option 1 is generally preferred because it gives you automatic type validation and keeps the code cleaner.

