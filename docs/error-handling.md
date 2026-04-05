# Error Handling

Errors in `oi` are treated as normal values, not as unexpected control flow jumps like Exceptions. This is a crucial element for ensuring AI-generated code handles edge cases rigorously.

## The Philosophy: No Exceptions
In many traditional languages, Exceptions cause hidden control flow diversions. A function that looks perfectly safe might suddenly throw an exception, bypassing normal logic.
`oi` embraces the philosophy that "Errors are just values."
- AI models can confidently cover every error path because the type checker enforces it.
- Human reviewers can trace error origins directly through the code flow.

## Result Types
Functions that can fail must return an Algebraic Data Type (ADT) called `Result`, typically `Result[T, E]`, where `T` is the success value and `E` is the error type.

```oi
fn read_file(path: String) -> Result[String, IOError] with io {
  -- Implementation
}
```

## Compiler-Enforced Handling
The `oi` compiler will not allow a `Result` type to be ignored. When you call a function that might fail, you must explicitly handle both the `Ok` and `Err` branches. This prevents fragile code that crashes on missing files or bad connections.

## Defining Error Types
Errors are distinct structured types (usually ADTs), making them highly descriptive.

```oi
type HttpError =
  | NotFound     { resource: String }
  | Unauthorized { reason: String }
  | ServerError  { cause: Error }
```

## The `?` Operator
To avoid the notorious boilerplate of languages like Go, `oi` provides the `?` operator. It bubbles the error up to the caller gracefully if it matches the function's own error return type.

## Error Mapping (`map_err`)
Often, internal module errors (like `DbError`) need to be mapped to external domain errors (like `HttpError`).

```oi
fn handle_request(req: Request) -> Result[Response, HttpError]
  with io, db
{
  -- Map an internal DB error to a public HTTP error
  let user = fetch_user(req.user_id)
    .map_err(|e| match e {
      DbError.NotFound => HttpError.NotFound { resource: "user" }
      other            => HttpError.ServerError { cause: other }
    })?

  let data = process(user, req.body)?
  Ok(Response.json(data, status: 200))
}
```

## Comparison with other approaches

- **Go**: Uses `if err != nil` everywhere, leading to massive boilerplate. `oi` avoids this via pattern matching and `?`.
- **Java**: Uses `throws`, but runtime unchecked exceptions escape the compiler's view. `oi` has zero un-tracked exceptions.
- **Python**: Relies totally on runtime `try/except`. AI struggles to predict which functions might throw `KeyError` or `ValueError`. `oi` eliminates this guessing game completely.
