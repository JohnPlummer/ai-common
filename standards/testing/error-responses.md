# Standard Error Response Format

Consistent JSON error response format for all API endpoints with standardized error codes and HTTP status mappings.

## Pattern

```json
{
  "code": "ERROR_CODE",
  "message": "Human-readable error message"
}
```

## Why Use This Pattern

- **Consistent**: All errors follow same structure
- **Machine readable**: Error codes enable automated handling
- **User friendly**: Clear messages for client display
- **Documented**: Error codes referenced in API documentation
- **Standardized**: HTTP status codes match error types

## HTTP Status Code Mapping

| Status | Error Code | Usage |
|--------|-----------|-------|
| 400 | INVALID_PARAMETER | Invalid query params, malformed requests |
| 404 | NOT_FOUND | Resource does not exist |
| 405 | METHOD_NOT_ALLOWED | Unsupported HTTP method |
| 500 | INTERNAL_ERROR | Server errors, database issues |

## Error Code Examples

### NOT_FOUND (404)

```json
{
  "code": "NOT_FOUND",
  "message": "Location 'brighton-uk' not found"
}

{
  "code": "NOT_FOUND",
  "message": "Activity '550e8400-e29b-41d4-a716-446655440000' not found"
}
```

### INVALID_PARAMETER (400)

```json
{
  "code": "INVALID_PARAMETER",
  "message": "Invalid category: food. Must be one of: food_drink, entertainment, culture, outdoors, shopping, wellness, sports"
}

{
  "code": "INVALID_PARAMETER",
  "message": "Invalid UUID format for parameter 'activityId'"
}
```

### INTERNAL_ERROR (500)

```json
{
  "code": "INTERNAL_ERROR",
  "message": "An internal error occurred while processing your request"
}
```

## Handler Implementation

```go
// backend/pkg/api/handlers/activity_handlers.go
type ErrorResponse struct {
    Code    string `json:"code"`
    Message string `json:"message"`
}

func (h *activityHandler) GetActivity(w http.ResponseWriter, r *http.Request) {
    activityID := chi.URLParam(r, "activityId")

    activity, err := h.service.GetActivity(r.Context(), activityID)
    if err != nil {
        if errors.Is(err, ErrNotFound) {
            h.writeError(w, http.StatusNotFound, "NOT_FOUND",
                fmt.Sprintf("Activity '%s' not found", activityID))
            return
        }

        h.logger.Error("failed to get activity", "error", err)
        h.writeError(w, http.StatusInternalServerError, "INTERNAL_ERROR",
            "An internal error occurred while processing your request")
        return
    }

    h.writeJSON(w, http.StatusOK, activity)
}

func (h *activityHandler) writeError(w http.ResponseWriter, status int, code, message string) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    json.NewEncoder(w).Encode(ErrorResponse{
        Code:    code,
        Message: message,
    })
}
```

## Parameter Validation

```go
func validateCategory(category string) error {
    validCategories := []string{
        "food_drink", "entertainment", "culture",
        "outdoors", "shopping", "wellness", "sports",
    }

    for _, valid := range validCategories {
        if category == valid {
            return nil
        }
    }

    return fmt.Errorf("invalid category: %s. Must be one of: %s",
        category, strings.Join(validCategories, ", "))
}
```

## Guidelines

1. **Use standard codes**: Stick to defined error codes (NOT_FOUND, INVALID_PARAMETER, INTERNAL_ERROR)
2. **Match status codes**: HTTP status must match error type semantics
3. **Clear messages**: Write messages that users/developers can act on
4. **No internal details**: Don't expose stack traces or internal errors to clients
5. **Log server errors**: Always log INTERNAL_ERROR details server-side

## Testing Error Responses

```go
var _ = Describe("Error Responses", func() {
    It("should return NOT_FOUND for missing activity", func() {
        req := httptest.NewRequest("GET", "/activities/invalid-id", nil)
        rec := httptest.NewRecorder()

        handler.GetActivity(rec, req)

        Expect(rec.Code).To(Equal(http.StatusNotFound))

        var response ErrorResponse
        json.Unmarshal(rec.Body.Bytes(), &response)
        Expect(response.Code).To(Equal("NOT_FOUND"))
        Expect(response.Message).To(ContainSubstring("not found"))
    })
})
```

## Related Patterns

- HTTP Middleware: See `.ai/project-standards/http-middleware.md`
- Error Classification: See `.ai/project-standards/error-classification.md`

## References

- Error Codes Documentation: `backend/docs/ERROR_CODES.md`
- OpenAPI Specification: `shared/api/openapi.yaml`
