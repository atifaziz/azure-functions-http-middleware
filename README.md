# azure-functions-http-middleware

An extensible middleware implementation for HTTP-triggered Azure Functions in .Net.  Lets you do stuff like this in your function:

```
public class MyFunction
{
    private readonly IMiddlewarePipeline pipeline;

    public MyPostFunction()
    {
        // This pipeline will:
        // 1. Extract correlation ID from request header,
        // 2. Validate the body payload contains all mandatory fields              
        // 3. Executes the logic for this Azure Function
        //
        // Any validation errors will result in a 400 Bad Request returned.
        
        this.pipeline = new MiddlewarePipeline();
        this.pipeline.UseCorrelationId(new string[] { "request-id" } )
                     .UseBodyValidation<BodyPayload>()
                     .Use(this.ExecuteAsync);
    }

    [FunctionName(nameof(MyFunction))]
    public async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequest req)
    {
        return await this.pipeline.RunAsync(req);
    }

    private async Task<IActionResult> ExecuteAsync(IHttpFunctionContext context)
    {
        // At this point, the body payload has been validated and correlation ID extracted from request headers.
        
        await Task.CompletedTask;

        dynamic payload = new
        {
            correlationId = context.CorrelationId,
            body = context.BodyModel
        };

        return new OkObjectResult(payload);
    }
}

public class BodyPayload
{
    [Required]
    public string Name { get; set; }

    public string Description { get; set; }
}
```

## Motivation

After having written several HTTP-triggered Azure Functions and writing the same cross-cutting concerns over and over for model validation, error handling, correlation IDs and such, it seemed appropriate to bundle all this into a package that can be re-used

This project was inspired by [this blog post](https://dasith.me/2018/01/20/using-azure-functions-httptrigger-as-web-api/) by Dasith Wijesiriwardena.

## Dependencies

Azure Functions 1.0.29
.Net Standard 2.0

