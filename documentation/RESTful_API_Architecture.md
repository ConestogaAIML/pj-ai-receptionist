# RESTful API Architecture

The application uses a standardized RESTful API architecture built on top of Django REST Framework (DRF). This architecture ensures a consistent response format across all endpoints and simplifies the creation of new API views.

The core of this architecture is defined in `main/viewsets.py`.

## Core Base Classes

All API views and viewsets should inherit from one of the following base classes to maintain consistency:

1. **`BaseModelViewSet`**: Inherits from DRF's `viewsets.ModelViewSet`. Use this for full CRUD operations tied to a specific Django model.
2. **`BaseViewSet`**: Inherits from DRF's `viewsets.ViewSet`. Use this for custom viewsets that don't directly map to a single model's standard CRUD.
3. **`BaseAPIView`**: Inherits from DRF's `APIView`. Use this for custom single-endpoint views (e.g., Webhooks). *Note: `BaseAPIView` allows unauthenticated access (`AllowAny`) by default.*

## Standardized Response Format

To maintain a consistent API contract with frontend and mobile clients, all base classes provide helper methods for generating responses. You should always use these methods instead of returning raw DRF `Response` objects.

### 1. Success Response
**Method:** `response_success(self, data, status_code=200, message=None, metadata=None)`

Used for successful operations. 

**Response Structure:**
```json
{
  "results": <data_payload>,
  "success": true,
  "status_code": 200,
  "message": "Optional success message",
  "metadata": { "optional": "metadata" }
}
```
*Note: The actual data payload is nested under the `results` key.*

### 2. Error Response
**Method:** `response_error(self, data=None, status_code=400, message=None, metadata=None)`

Used for client or server errors (4xx or 5xx status codes).

**Response Structure:**
```json
{
  "data": <error_data_or_null>,
  "success": false,
  "status_code": 400,
  "message": "Optional error message",
  "metadata": { "optional": "metadata" }
}
```

### 3. Unauthorized Response
**Method:** `response_unauthorized(self, data, status_code=401, message=None, metadata=None)`

Used when authentication or authorization fails. The response structure is identical to `response_error`.

### 4. Paginated Response
**Method:** `get_paginated_response(self, data, status_code=200, message=None, metadata=None)`

Used for returning paginated lists. It automatically intercepts DRF's built-in paginator and injects the standardized fields (`success`, `status_code`, `message`, `metadata`) into the top level of the paginated response.

## Best Practices

- **Always use the helper methods:** When writing a custom action in a viewset or a custom `APIView`, always return `self.response_success(...)` or `self.response_error(...)`.
- **Exception Catching:** The `BaseModelViewSet.response_success` method includes a built-in try-except block that automatically catches exceptions and falls back to `response_error` with the exception message.
- **Automatic Wrapping:** The `BaseModelViewSet` overrides the default `retrieve` method to ensure that standard object retrieval automatically conforms to the `response_success` format.
