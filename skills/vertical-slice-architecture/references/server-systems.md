# Vertical Slice Architecture Agent Skill

## Navigation

- Operating definition: section 1
- When to use this skill: section 2
- Mental model and core principles: sections 4-5
- Slice granularity and folder structures: sections 7-8
- Dependency boundaries: section 10
- CQRS, mediator, and framework integration: sections 11-14
- Domain modeling, persistence, validation, authorization, and errors: sections 15-19
- Testing, observability, and scalability: sections 21-23
- Migration, review checklist, and implementation checklist: sections 28-30
- Examples and templates: sections 32-40

## Skill metadata

**Skill name:** Vertical Slice Architecture  
**Primary use:** Help agents design, review, refactor, and implement software using Vertical Slice Architecture (VSA).  
**Audience:** AI coding agents, software architects, senior engineers, and reviewers.  
**Applies to:** Backend APIs, full-stack web apps, modular monoliths, microservices, serverless functions, CQRS systems, event-driven systems, and feature-oriented frontends.  
**Not limited to:** .NET, MediatR, CQRS, Minimal APIs, or Clean Architecture. Those are common implementation choices, not the definition of the architecture.

---

## 1. Operating definition

Vertical Slice Architecture organizes software around **business capabilities, user-facing use cases, or request workflows**, rather than around horizontal technical layers such as `Controllers`, `Services`, `Repositories`, `Validators`, `DTOs`, and `Models`.

A vertical slice should contain, as locally as practical, everything needed to fulfill a single behavior:

- Entry point: HTTP endpoint, page action, message consumer, CLI command, scheduled job, GraphQL resolver, UI action, etc.
- Input contract: request DTO, command, query, form model, event payload.
- Validation and authorization rules specific to that behavior.
- Handler or use-case logic.
- Mapping and response contract.
- Persistence or query logic needed only by that behavior.
- Tests for the behavior.
- Documentation or example request, when useful.
- UI component or client-side action, when the team slices full-stack.

The core rule:

> **Group things that change together. Keep feature-specific behavior close. Share only what is stable, intentional, and truly cross-cutting.**

---

## 2. When to use this skill

Use this skill when the user asks an agent to:

- Design or evaluate an application architecture.
- Refactor a layered project toward feature-oriented organization.
- Create folder structures for APIs, services, or full-stack apps.
- Decide where commands, queries, handlers, validators, DTOs, entities, repositories, or tests should live.
- Integrate CQRS, MediatR, Wolverine, Minimal APIs, FastEndpoints, NestJS, Spring, Rails, Django, Express, or similar frameworks with feature folders.
- Review whether a design violates slice boundaries.
- Decide what should be shared versus duplicated.
- Improve testability, modularity, cohesion, and maintainability.
- Explain tradeoffs between Clean Architecture, Hexagonal Architecture, modular monoliths, microservices, and Vertical Slice Architecture.

Do **not** use this skill as a mechanical template generator only. VSA is primarily a design approach based on cohesion and coupling. Folder structure is a consequence.

---

## 3. High-level source synthesis

This skill is synthesized from several commonly cited and high-quality sources:

- Jimmy Bogard, "Vertical Slice Architecture"  
  https://www.jimmybogard.com/vertical-slice-architecture/
- Derek Comartin, "Restructuring to a Vertical Slice Architecture"  
  https://codeopinion.com/restructuring-to-a-vertical-slice-architecture/
- Derek Comartin, "Vertical Slice Architecture Myths You Need To Know!"  
  https://codeopinion.com/vertical-slice-architecture-myths-you-need-to-know/
- Derek Comartin, "Vertical Slice Architecture isn't technical"  
  https://codeopinion.com/vertical-slice-architecture-isnt-technical/
- Milan Jovanovic, "Vertical Slice Architecture"  
  https://www.milanjovanovic.tech/blog/vertical-slice-architecture
- Milan Jovanovic, "Vertical Slice Architecture: Structuring Vertical Slices"  
  https://www.milanjovanovic.tech/blog/vertical-slice-architecture-structuring-vertical-slices
- Microsoft Learn, "Feature Slices for ASP.NET Core MVC"  
  https://learn.microsoft.com/en-us/archive/msdn-magazine/2016/september/asp-net-core-feature-slices-for-asp-net-core-mvc
- Microsoft Learn, "CQRS pattern"  
  https://learn.microsoft.com/en-us/azure/architecture/patterns/cqrs
- Microsoft Learn, ASP.NET Core Minimal APIs and endpoint filters  
  https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis
  https://learn.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis/min-api-filters
- MediatR GitHub repository  
  https://github.com/LuckyPennySoftware/MediatR
- Steve Smith, "API Feature Folders"  
  https://ardalis.com/api-feature-folders/
- Tim G. Thomas, "Feature Folders in ASP.NET MVC"  
  https://timgthomas.com/2013/10/feature-folders-in-asp-net-mvc/
- Robert C. Martin, "The Clean Architecture"  
  https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html
- Wolverine documentation, "Vertical Slice Architecture"  
  https://wolverinefx.net/tutorials/vertical-slice-architecture

---

## 4. Mental model

### 4.1 Horizontal layering

A traditional layered application often looks like this:

```text
src/
  Controllers/
    OrdersController.cs
    CustomersController.cs
  Services/
    OrderService.cs
    CustomerService.cs
  Repositories/
    OrderRepository.cs
    CustomerRepository.cs
  DTOs/
    CreateOrderRequest.cs
    OrderResponse.cs
  Validators/
    CreateOrderRequestValidator.cs
  Mapping/
    OrderProfile.cs
  Domain/
    Order.cs
    Customer.cs
```

When implementing `CreateOrder`, an agent may need to touch:

```text
Controllers/OrdersController.cs
Services/OrderService.cs
Repositories/OrderRepository.cs
DTOs/CreateOrderRequest.cs
DTOs/OrderResponse.cs
Validators/CreateOrderRequestValidator.cs
Mapping/OrderProfile.cs
Domain/Order.cs
```

The problem is not that layers are always wrong. The problem is that the **unit of change** is a feature, while the **unit of organization** is a technical role. This creates navigation overhead and tends to encourage global abstractions.

### 4.2 Vertical slicing

A vertical slice version might look like this:

```text
src/
  Features/
    Orders/
      CreateOrder/
        CreateOrderEndpoint.cs
        CreateOrderCommand.cs
        CreateOrderHandler.cs
        CreateOrderValidator.cs
        CreateOrderResponse.cs
        CreateOrderTests.cs
      GetOrder/
        GetOrderEndpoint.cs
        GetOrderQuery.cs
        GetOrderHandler.cs
        GetOrderResponse.cs
        GetOrderTests.cs
    Customers/
      RegisterCustomer/
      GetCustomerProfile/
  Shared/
    Auth/
    Persistence/
    Messaging/
    Clock.cs
```

Now the unit of organization and the unit of change are closer.

---

## 5. Core principles

### Principle 1: Slice by business capability, not technical artifact

Prefer:

```text
Features/Orders/CreateOrder
Features/Orders/CancelOrder
Features/Orders/GetOrderSummary
Features/Payments/CapturePayment
Features/Shipping/SchedulePickup
```

Avoid:

```text
Controllers/
Services/
Repositories/
Validators/
DTOs/
```

This does not mean a slice cannot contain a class named `Validator` or `Handler`. It means those files are grouped by the behavior they support.

### Principle 2: Maximize cohesion inside a slice

A good slice has high internal cohesion. The endpoint, request, validation, authorization, use-case logic, data access, and response model are all there because they serve one behavior.

Ask:

- Would these files normally change in the same pull request?
- Does reading this folder explain the behavior end to end?
- Can a developer modify this behavior without navigating the whole solution?
- Can a test exercise this behavior through its real entry point?

### Principle 3: Minimize coupling between slices

Slices should not casually reach into each other.

Bad:

```csharp
// Inside Payments/CapturePayment
await _shippingSchedulePickupHandler.Handle(...);
```

Better:

```csharp
// Publish a domain/integration event
await _eventBus.Publish(new PaymentCaptured(orderId));
```

Or orchestrate explicitly:

```csharp
// Application workflow/saga coordinates multiple capabilities.
await _paymentGateway.Capture(orderId);
await _messageBus.Publish(new PaymentCaptured(orderId));
```

Rule:

> A slice may depend on stable shared primitives, shared infrastructure contracts, and domain concepts. It should not depend on another slice's private request, handler, validator, or DTO.

### Principle 4: Couple along the axis of change

If a validator, DTO, query, and endpoint change together, keep them together.

If a domain invariant is used by many behaviors and represents the actual business model, move it to the domain model or shared capability module.

If a utility is stable and generic, put it in shared infrastructure.

If code is merely duplicated twice, duplication may be cheaper than premature abstraction.

### Principle 5: Use the simplest implementation per slice

A read-only reporting slice may use direct SQL or Dapper.

A complex command slice may use an aggregate, domain events, transaction boundary, and optimistic concurrency.

A CRUD admin slice may use transaction script style.

Do not force every slice to use the same abstraction level.

### Principle 6: Cross-cutting concerns are centralized by policy, not by feature logic

Examples:

- Logging
- Metrics
- Tracing
- Authentication
- Authorization framework integration
- Validation pipeline
- Transaction behavior
- Retry behavior
- Idempotency middleware
- Correlation IDs
- Exception mapping
- Outbox dispatch
- Unit of Work commit

These concerns may be implemented through framework middleware, endpoint filters, mediator pipeline behaviors, decorators, interceptors, or base endpoint conventions.

### Principle 7: Do not confuse physical deployment with slicing

Vertical Slice Architecture can be used in:

- A monolith
- A modular monolith
- A microservice
- A serverless function app
- A frontend SPA
- A full-stack app
- A background worker

It does not automatically imply microservices or independent deployment.

### Principle 8: Do not confuse VSA with "one file per feature"

A slice can be one file for a small feature, but it does not have to be.

Good:

```text
CreateOrder.cs
```

Also good:

```text
CreateOrder/
  Endpoint.cs
  Command.cs
  Handler.cs
  Validator.cs
  Response.cs
  Mapping.cs
  Tests.cs
```

Bad:

```text
CreateOrder/
  Controllers/
  Services/
  Repositories/
  Validators/
```

That recreates layers inside every slice.

---

## 6. Vocabulary

### Slice

A cohesive unit of behavior. Often one command, one query, one endpoint, one page action, one message consumer, or one workflow step.

### Feature

A user-visible capability or product behavior. A feature can contain multiple slices.

Example:

```text
Feature: Orders
Slices:
  CreateOrder
  GetOrder
  CancelOrder
  ListCustomerOrders
  AddOrderNote
```

### Capability

A broader business ability. It may map to a module or bounded context.

Example:

```text
Capability: Fulfillment
Features:
  Picking
  Packing
  Dispatch
  Delivery tracking
```

### Module

A larger boundary that owns a cluster of related slices. In a modular monolith, modules are often stronger boundaries than individual slices.

Example:

```text
Modules/
  Ordering/
    Features/
  Payments/
    Features/
  Shipping/
    Features/
```

### Bounded context

A Domain-Driven Design boundary where a model has a consistent meaning. A bounded context may contain many modules, features, and slices.

### Shared kernel

A deliberately small shared model used across boundaries. Treat it as expensive to change.

### Public contract

A stable interface, event, API, or package that another module or external system may consume.

---

## 7. Slice granularity

### Good slice sizes

A good slice is usually one of these:

1. One HTTP request
2. One command
3. One query
4. One page interaction
5. One background job behavior
6. One message handler
7. One GraphQL resolver behavior
8. One CLI command
9. One workflow step

Examples:

```text
Orders/CreateOrder
Orders/CancelOrder
Orders/GetOrderDetails
Orders/ListOrders
Payments/CapturePayment
Payments/RefundPayment
Shipping/SchedulePickup
Reports/GetMonthlyRevenue
```

### Too coarse

```text
Orders/OrderService
Orders/ManageOrders
Orders/AllOrderOperations
```

Symptoms:

- Folder contains many unrelated request types.
- Handler has many `if` or `switch` branches.
- Tests are hard to name.
- Multiple teams constantly edit the same files.
- Authorization and validation rules are mixed.

### Too fine

```text
Orders/CreateOrder/ValidateOrderName
Orders/CreateOrder/ValidateOrderAddress
Orders/CreateOrder/SaveOrderLine
Orders/CreateOrder/SendOrderEmail
```

Symptoms:

- Folder structure becomes noisy.
- Agents create many micro-handlers with no real independence.
- A single user action requires jumping across many slices.
- Business transaction boundaries become unclear.

### Practical rule

Start with one slice per externally observable behavior. Refactor when the slice becomes hard to understand, test, or modify.

---

## 8. Recommended folder structures

### 8.1 Single-project backend API

```text
src/
  Api/
    Program.cs
    CompositionRoot.cs
    Features/
      Orders/
        CreateOrder/
          Endpoint.cs
          Command.cs
          Handler.cs
          Validator.cs
          Response.cs
        GetOrder/
          Endpoint.cs
          Query.cs
          Handler.cs
          Response.cs
      Payments/
        CapturePayment/
        RefundPayment/
    Shared/
      Auth/
      Errors/
      Persistence/
      Messaging/
      Time/
      Validation/
  Tests/
    Api.Tests/
      Features/
        Orders/
          CreateOrderTests.cs
```

Use when:

- App is small to medium.
- Team wants low ceremony.
- Module boundaries are not yet mature.

### 8.2 Modular monolith with vertical slices inside modules

```text
src/
  Modules/
    Ordering/
      Ordering.csproj
      Domain/
        Order.cs
        OrderLine.cs
      Features/
        CreateOrder/
        CancelOrder/
        GetOrder/
      Infrastructure/
        OrderingDbContext.cs
      Contracts/
        OrderPlaced.cs
    Payments/
      Payments.csproj
      Domain/
      Features/
      Infrastructure/
      Contracts/
    Shipping/
      Shipping.csproj
      Domain/
      Features/
      Infrastructure/
      Contracts/
  Host/
    ApiHost.csproj
  SharedKernel/
    SharedKernel.csproj
```

Use when:

- System has multiple business capabilities.
- Team needs stronger compile-time boundaries.
- Modules may later become services.
- You want architecture tests to prevent cross-module access.

### 8.3 Full-stack slice

```text
src/
  features/
    orders/
      create-order/
        ui/
          CreateOrderForm.tsx
          useCreateOrder.ts
        api/
          route.ts
          schema.ts
          handler.ts
        domain/
          order-rules.ts
        tests/
          create-order.spec.ts
          create-order.api.test.ts
      get-order/
        ui/
        api/
        tests/
  shared/
    ui/
    auth/
    http/
    db/
```

Use when:

- The team controls frontend and backend.
- You want product features to be visible in the repository.
- UI and API changes normally happen together.

### 8.4 Serverless feature folders

```text
src/
  functions/
    orders/
      create-order/
        function.ts
        command.ts
        validator.ts
        handler.ts
        response.ts
        tests.ts
      cancel-order/
    payments/
      payment-webhook/
  shared/
    event-bus/
    db/
    observability/
```

Use when:

- Each function maps naturally to a behavior.
- Deployment or scaling is per function.
- Infrastructure-as-code can be grouped with the slice.

### 8.5 CQRS-oriented structure

```text
src/
  Features/
    Orders/
      Commands/
        CreateOrder/
          Command.cs
          Handler.cs
          Validator.cs
          Endpoint.cs
        CancelOrder/
      Queries/
        GetOrderDetails/
          Query.cs
          Handler.cs
          Endpoint.cs
          Response.cs
        ListOrders/
```

Use when:

- The team strongly separates reads and writes.
- Commands and queries are conceptually important.
- You still want feature-first grouping.

Avoid making `Commands/` and `Queries/` the top-level folders for the whole application. That usually recreates layers.

---

## 9. File naming conventions

### Compact static-class style

Good for .NET Minimal APIs:

```text
Features/Products/CreateProduct.cs
```

Inside:

```csharp
public static class CreateProduct
{
    public sealed record Request(string Name, decimal Price);
    public sealed record Response(Guid Id, string Name, decimal Price);

    public sealed class Validator : AbstractValidator<Request> { }

    public static async Task<Results<Ok<Response>, ValidationProblem>> Handle(
        Request request,
        AppDbContext db,
        CancellationToken cancellationToken)
    {
        // implementation
    }

    public static void Map(IEndpointRouteBuilder app)
    {
        app.MapPost("/products", Handle);
    }
}
```

Pros:

- Very local.
- Easy to read for small slices.
- Low ceremony.

Cons:

- Large slices can become too dense.
- Harder to reuse nested types from tests in some languages.
- Can produce very large files when overused.

### Multi-file slice folder

```text
CreateProduct/
  Endpoint.cs
  Request.cs
  Response.cs
  Validator.cs
  Handler.cs
  Tests.cs
```

Pros:

- Scales better for complex slices.
- Clear separation without moving files away from the feature.
- Easier for agents to edit one concern at a time.

Cons:

- More files.
- Can become repetitive if every tiny slice uses all files.

### Hybrid convention

Use one file for simple slices, folder for complex slices.

```text
Products/
  GetProduct.cs
  ListProducts.cs
  CreateProduct/
    Endpoint.cs
    Command.cs
    Handler.cs
    Validator.cs
```

Rule:

> Prefer locality over uniformity. Use uniformity only where it reduces confusion.

---

## 10. Dependency boundary rules

### 10.1 Default allowed dependencies

A slice may depend on:

- Language/runtime libraries.
- Framework entry-point abstractions, if they are unavoidable.
- Cross-cutting infrastructure abstractions, such as `IClock`, `ICurrentUser`, `IEventBus`.
- Persistence context or query client, if that is the selected tradeoff.
- Domain model or aggregate owned by the same module.
- Shared kernel primitives, such as `Money`, `EmailAddress`, `Result`, `Error`.
- Contracts from another module, if they are explicitly public contracts.
- Message/event types intended for integration.

### 10.2 Default forbidden dependencies

A slice should not depend on:

- Another slice's handler.
- Another slice's request or response DTO.
- Another slice's validator.
- Another slice's private repository.
- Another slice's endpoint.
- Another slice's private mapping profile.
- UI components from another feature unless they are moved to shared UI.
- Query models that were designed only for another endpoint.

### 10.3 Public versus private code

Use naming, folders, language visibility, or packages to mark boundaries.

```text
Ordering/
  Features/
    CreateOrder/       # private to feature
    CancelOrder/       # private to feature
  Domain/              # shared inside Ordering
  Contracts/           # public outside Ordering
    OrderPlaced.cs
    OrderCanceled.cs
  PublicApi/           # explicit application facade if needed
```

Rule:

> Other modules may use `Contracts/` and `PublicApi/`, but not `Features/*`.

### 10.4 When a slice needs another slice

Do not call the other slice directly. Choose one:

1. **Extract shared domain behavior**
   - Move the common rule to an aggregate, domain service, or value object.

2. **Extract shared query**
   - If two endpoints need the same read model, create a shared read model in the module, not in one slice.

3. **Create a public application service**
   - Use only for stable cross-slice capability, not convenience.

4. **Publish an event**
   - For asynchronous workflows or integration.

5. **Create an orchestrating workflow**
   - For a business process that coordinates multiple slices.

6. **Merge slices**
   - If they always change together and are not independently meaningful.

### 10.5 Shared logic decision table

| Situation | Recommended action |
|---|---|
| Same two-line guard appears in two slices | Duplicate until a third use or clear semantic name appears |
| Same domain invariant appears in multiple command slices | Move into aggregate, entity, value object, or domain service |
| Same validation rule appears for the same concept | Move into reusable validator extension or value object |
| Same query projection appears in multiple read slices | Extract a read model or query helper inside the module |
| Same HTTP error mapping appears everywhere | Use middleware, endpoint filter, or pipeline behavior |
| Same transaction logic appears in commands | Use pipeline behavior, decorator, or unit-of-work middleware |
| Same call to another slice appears in many places | Introduce event, workflow, or explicit module API |
| Same utility is generic and stable | Put in `Shared` or common library |
| Same utility depends on business meaning | Put in domain/module, not global shared |

---

## 11. CQRS integration

### 11.1 CQRS definition

Command Query Responsibility Segregation separates operations that mutate state from operations that read state.

In VSA, CQRS often fits naturally:

```text
Orders/CreateOrder      # command
Orders/CancelOrder      # command
Orders/GetOrderDetails  # query
Orders/ListOrders       # query
```

### 11.2 CQRS is optional

VSA does not require CQRS.

A simple CRUD slice can still be vertical:

```text
Products/UpdateProduct
  Endpoint.cs
  Request.cs
  Handler.cs
  Validator.cs
```

But command/query naming often improves clarity.

### 11.3 Command slice

A command slice:

- Changes state.
- Has a clear business intent.
- Usually needs validation.
- Usually needs authorization.
- Usually has a transaction boundary.
- May raise domain or integration events.
- May need idempotency.
- Often returns an ID, status, or updated representation.

Example names:

```text
CreateOrder
CancelOrder
ApproveInvoice
CapturePayment
AssignTicket
ScheduleShipment
```

### 11.4 Query slice

A query slice:

- Reads state.
- Does not mutate state.
- Can use optimized projection.
- Can bypass aggregates if no business invariant is enforced.
- Can use direct SQL, view, cache, search index, document projection, or ORM projection.
- Usually returns a DTO shaped for that specific use case.

Example names:

```text
GetOrderDetails
ListCustomerOrders
SearchProducts
GetDashboardMetrics
ExportMonthlyRevenue
```

### 11.5 Separate models per operation

Do not reuse write entities as API responses by default.

Good:

```csharp
public sealed record OrderDetailsResponse(
    Guid Id,
    string Status,
    decimal Total,
    IReadOnlyList<OrderLineResponse> Lines);
```

Risky:

```csharp
public async Task<Order> GetOrder(Guid id) => await db.Orders.FindAsync(id);
```

Reasons:

- API response shape differs from persistence shape.
- Some fields must be hidden.
- Read models can denormalize.
- Query performance differs from write model needs.
- Versioning API contracts is easier when DTOs are explicit.

### 11.6 Separate physical stores only when needed

CQRS does not require separate databases.

Levels:

1. Same entity model, separate command/query handlers.
2. Same database, separate read DTO projections.
3. Same database, separate read tables or views.
4. Separate read store, updated asynchronously.
5. Event sourcing with projections.

Pick the simplest level that solves the problem.

---

## 12. MediatR, mediator, and pipeline integration

### 12.1 What mediator gives you

A mediator library can provide:

- Request-response dispatch.
- Command/query handlers.
- Notifications/events.
- Pipeline behaviors around handlers.
- Decoupling of endpoint from handler implementation.
- Test seam for handlers.

### 12.2 What mediator does not give you

Mediator does not automatically give you good architecture. Agents must still design:

- Slice boundaries.
- Business capability boundaries.
- Shared code policy.
- Domain model ownership.
- Transaction boundaries.
- Error handling.
- Tests.
- Observability.

### 12.3 Typical .NET slice with MediatR

```text
Features/Orders/CreateOrder/
  Endpoint.cs
  Command.cs
  Handler.cs
  Validator.cs
  Response.cs
```

```csharp
public sealed record CreateOrderCommand(
    Guid CustomerId,
    IReadOnlyList<CreateOrderLine> Lines
) : IRequest<CreateOrderResponse>;

public sealed record CreateOrderLine(Guid ProductId, int Quantity);

public sealed record CreateOrderResponse(Guid OrderId);
```

```csharp
public sealed class CreateOrderValidator : AbstractValidator<CreateOrderCommand>
{
    public CreateOrderValidator()
    {
        RuleFor(x => x.CustomerId).NotEmpty();
        RuleFor(x => x.Lines).NotEmpty();
        RuleForEach(x => x.Lines).ChildRules(line =>
        {
            line.RuleFor(x => x.ProductId).NotEmpty();
            line.RuleFor(x => x.Quantity).GreaterThan(0);
        });
    }
}
```

```csharp
public sealed class CreateOrderHandler
    : IRequestHandler<CreateOrderCommand, CreateOrderResponse>
{
    private readonly AppDbContext _db;
    private readonly ICurrentUser _currentUser;
    private readonly IEventBus _events;

    public CreateOrderHandler(
        AppDbContext db,
        ICurrentUser currentUser,
        IEventBus events)
    {
        _db = db;
        _currentUser = currentUser;
        _events = events;
    }

    public async Task<CreateOrderResponse> Handle(
        CreateOrderCommand command,
        CancellationToken cancellationToken)
    {
        var order = Order.Create(
            customerId: command.CustomerId,
            createdBy: _currentUser.UserId);

        foreach (var line in command.Lines)
        {
            order.AddLine(line.ProductId, line.Quantity);
        }

        _db.Orders.Add(order);
        await _db.SaveChangesAsync(cancellationToken);

        await _events.Publish(
            new OrderPlaced(order.Id),
            cancellationToken);

        return new CreateOrderResponse(order.Id);
    }
}
```

```csharp
public static class CreateOrderEndpoint
{
    public static IEndpointRouteBuilder MapCreateOrder(this IEndpointRouteBuilder app)
    {
        app.MapPost("/orders", async (
            CreateOrderCommand command,
            ISender sender,
            CancellationToken cancellationToken) =>
        {
            var response = await sender.Send(command, cancellationToken);
            return Results.Created($"/orders/{response.OrderId}", response);
        })
        .WithTags("Orders")
        .RequireAuthorization("CanCreateOrders");

        return app;
    }
}
```

### 12.4 Pipeline behaviors

Use pipeline behaviors for policies that are orthogonal to specific slices.

Common behaviors:

```text
ValidationBehavior<TRequest, TResponse>
AuthorizationBehavior<TRequest, TResponse>
LoggingBehavior<TRequest, TResponse>
PerformanceBehavior<TRequest, TResponse>
TransactionBehavior<TRequest, TResponse>
IdempotencyBehavior<TRequest, TResponse>
ExceptionMappingBehavior<TRequest, TResponse>
```

Rules:

- Keep business logic out of generic behaviors.
- Make behavior order explicit.
- Avoid hidden magic that agents cannot reason about.
- Ensure integration tests cover behavior registration.
- Do not put feature-specific validation into a global behavior. The behavior should only execute validators defined per request.

### 12.5 Pipeline behavior order

A typical order:

```text
1. Correlation/log scope
2. Authorization
3. Idempotency
4. Validation
5. Transaction
6. Handler
7. Domain event dispatch/outbox
8. Response logging/metrics
9. Exception mapping around the chain
```

Exact order depends on requirements.

Important examples:

- Authorization before validation can avoid leaking validation details to unauthorized callers.
- Validation before transaction avoids opening unnecessary transactions.
- Idempotency before transaction can return cached command results.
- Outbox dispatch should be consistent with the transaction boundary.

---

## 13. ASP.NET Core Minimal APIs and endpoint-first slices

### 13.1 Minimal API feature structure

```text
Features/
  Products/
    CreateProduct.cs
    GetProduct.cs
    ListProducts.cs
```

```csharp
public static class CreateProduct
{
    public sealed record Request(string Name, decimal Price);
    public sealed record Response(Guid Id, string Name, decimal Price);

    public static async Task<Results<Created<Response>, ValidationProblem>> Handle(
        Request request,
        AppDbContext db,
        CancellationToken cancellationToken)
    {
        var product = new Product(request.Name, request.Price);

        db.Products.Add(product);
        await db.SaveChangesAsync(cancellationToken);

        var response = new Response(product.Id, product.Name, product.Price);
        return TypedResults.Created($"/products/{product.Id}", response);
    }

    public static void Map(IEndpointRouteBuilder group)
    {
        group.MapPost("/", Handle)
            .WithName("CreateProduct")
            .WithSummary("Create a product");
    }
}
```

```csharp
public static class ProductEndpoints
{
    public static IEndpointRouteBuilder MapProductEndpoints(this IEndpointRouteBuilder app)
    {
        var group = app.MapGroup("/products")
            .WithTags("Products")
            .RequireAuthorization();

        CreateProduct.Map(group);
        GetProduct.Map(group);
        ListProducts.Map(group);

        return app;
    }
}
```

### 13.2 Endpoint filters

Endpoint filters are useful for:

- Validation.
- Logging.
- API version policy.
- Request shaping.
- Response mapping.
- Common endpoint-specific concerns.

Do not put complex business logic in filters.

### 13.3 Route groups

Route groups let agents organize endpoint registration around features:

```csharp
var orders = app.MapGroup("/orders")
    .WithTags("Orders")
    .RequireAuthorization();

CreateOrder.Map(orders);
CancelOrder.Map(orders);
GetOrder.Map(orders);
```

This keeps route registration aligned with slice organization.

---

## 14. Framework integration patterns

### 14.1 .NET with MediatR

Best fit when:

- You want command/query handlers.
- You want pipeline behaviors.
- You prefer endpoints thinly dispatching requests.
- You want a familiar CQRS style.

Structure:

```text
Features/Orders/CreateOrder/
  Endpoint.cs
  Command.cs
  Handler.cs
  Validator.cs
```

### 14.2 .NET without MediatR

Best fit when:

- You want fewer abstractions.
- You are comfortable injecting dependencies into endpoints.
- Your app is small to medium.
- You want straightforward call graphs.

Structure:

```text
Features/Orders/CreateOrder.cs
```

Handler can be a static method or class.

### 14.3 Wolverine or message-first .NET

Best fit when:

- You use CQRS with messaging.
- You want command handlers, local queues, durable messaging, or outbox support.
- You want to reduce explicit mediator ceremony.

Structure:

```text
Orders/
  CreateOrder/
    CreateOrder.cs
    CreateOrderHandler.cs
```

### 14.4 FastEndpoints or REPR-style APIs

Best fit when:

- You like Request-EndPoint-Response.
- You want one class or folder per endpoint.
- You want endpoint classes rather than controllers.

Structure:

```text
Features/Orders/CreateOrder/
  Request.cs
  Endpoint.cs
  Response.cs
```

### 14.5 Java Spring Boot

Avoid package-by-layer:

```text
com.example.app.controller
com.example.app.service
com.example.app.repository
```

Prefer package-by-feature:

```text
com.example.app.orders.create
  CreateOrderController.java
  CreateOrderRequest.java
  CreateOrderHandler.java
  CreateOrderValidator.java
  CreateOrderResponse.java

com.example.app.orders.get
  GetOrderController.java
  GetOrderQuery.java
  GetOrderHandler.java
```

For larger systems:

```text
com.example.app.ordering
  domain
  features.createorder
  features.cancelorder
  infrastructure
  contracts
```

### 14.6 NestJS

Avoid only technical modules like `controllers/`, `services/`, and `dto/`.

Prefer:

```text
src/
  features/
    orders/
      create-order/
        create-order.controller.ts
        create-order.command.ts
        create-order.handler.ts
        create-order.dto.ts
        create-order.spec.ts
      get-order/
    payments/
  shared/
    db/
    auth/
```

Nest modules can align with business modules:

```text
orders.module.ts
payments.module.ts
shipping.module.ts
```

### 14.7 Express or Fastify

```text
src/
  features/
    orders/
      create-order/
        route.ts
        schema.ts
        handler.ts
        response.ts
        test.ts
      get-order/
  shared/
    db.ts
    auth.ts
```

Example:

```ts
// features/orders/create-order/route.ts
import { FastifyInstance } from "fastify";
import { createOrderSchema } from "./schema";
import { createOrderHandler } from "./handler";

export async function registerCreateOrder(app: FastifyInstance) {
  app.post("/orders", { schema: createOrderSchema }, createOrderHandler);
}
```

```ts
// features/orders/create-order/handler.ts
import { db } from "../../../shared/db";

export async function createOrderHandler(request, reply) {
  const { customerId, lines } = request.body;

  const order = await db.transaction(async (tx) => {
    const created = await tx.order.create({ data: { customerId } });

    await tx.orderLine.createMany({
      data: lines.map((line) => ({
        orderId: created.id,
        productId: line.productId,
        quantity: line.quantity,
      })),
    });

    return created;
  });

  return reply.code(201).send({ orderId: order.id });
}
```

### 14.8 Django

Django often encourages app-level modules. Align apps with business capabilities, then use slices inside apps.

```text
orders/
  features/
    create_order/
      views.py
      forms.py
      service.py
      serializers.py
      tests.py
    cancel_order/
  models.py
  urls.py
  contracts.py
```

Avoid dumping everything into global `views.py` and `serializers.py`.

### 14.9 Rails

Rails convention is highly layered by default. VSA can still be approximated:

```text
app/
  features/
    orders/
      create_order/
        command.rb
        validator.rb
        presenter.rb
        spec.rb
  controllers/
    orders_controller.rb
```

Controller actions dispatch into feature commands. Use this when Rails service objects become a service graveyard.

### 14.10 React or frontend feature slicing

```text
src/
  features/
    checkout/
      place-order/
        PlaceOrderForm.tsx
        usePlaceOrder.ts
        schema.ts
        api.ts
        tests.tsx
    orders/
      order-details/
        OrderDetailsPage.tsx
        query.ts
        components/
  shared/
    ui/
    auth/
    http/
    formatting/
```

Frontend guidance:

- Shared UI components go in `shared/ui`.
- Feature-specific components stay in the feature.
- API calls that serve only one feature stay in the feature.
- Cross-feature state should be deliberate, not accidental.
- Avoid global `components/`, `hooks/`, and `utils/` dumping grounds.

### 14.11 GraphQL

GraphQL schema often cuts across use cases. Use slice folders for resolver behavior:

```text
features/
  orders/
    queries/
      order-details/
        resolver.ts
        projection.ts
        response.ts
    mutations/
      create-order/
        resolver.ts
        input.ts
        handler.ts
        validator.ts
```

Avoid:

- One giant `OrderResolver`.
- Shared GraphQL types that expose persistence models.
- Mutations that reuse query DTOs.

### 14.12 gRPC

```text
Features/
  Orders/
    CreateOrder/
      CreateOrderGrpcEndpoint.cs
      CreateOrderHandler.cs
      CreateOrderValidator.cs
    GetOrder/
```

Generated protocol types may live in a contracts project, but behavior stays in slices.

### 14.13 Background jobs

```text
Features/
  Billing/
    GenerateMonthlyInvoices/
      Job.cs
      Handler.cs
      Query.cs
      Tests.cs
  Emails/
    SendWelcomeEmail/
      Consumer.cs
      Template.cs
      Tests.cs
```

A job is an entry point like an endpoint. Treat it as a slice.

---

## 15. Domain modeling in VSA

### 15.1 Transaction script is acceptable for simple behavior

For simple CRUD or reporting:

```csharp
public async Task<Response> Handle(Query query)
{
    return await _db.Orders
        .Where(o => o.Id == query.OrderId)
        .Select(o => new Response(o.Id, o.Status, o.Total))
        .SingleAsync();
}
```

This is fine when no domain invariant is being enforced.

### 15.2 Push domain logic down when invariants appear

Bad:

```csharp
if (order.Status == "Cancelled")
    throw new InvalidOperationException();

if (order.Lines.Count == 0)
    throw new InvalidOperationException();

order.Status = "Submitted";
```

Better:

```csharp
order.Submit();
```

Inside domain:

```csharp
public void Submit()
{
    if (Status == OrderStatus.Cancelled)
        throw new DomainException("Cancelled orders cannot be submitted.");

    if (!_lines.Any())
        throw new DomainException("An order requires at least one line.");

    Status = OrderStatus.Submitted;
    AddDomainEvent(new OrderSubmitted(Id));
}
```

### 15.3 Domain model placement options

#### Option A: Domain inside module

```text
Modules/Ordering/
  Domain/
    Order.cs
    OrderLine.cs
    OrderStatus.cs
  Features/
    CreateOrder/
    SubmitOrder/
```

Best for complex modules.

#### Option B: Domain inside feature

```text
Features/Orders/CreateOrder/
  OrderDraft.cs
  CreateOrderHandler.cs
```

Best when the model is used only by that slice.

#### Option C: Shared kernel

```text
SharedKernel/
  Money.cs
  EmailAddress.cs
  TenantId.cs
```

Use sparingly.

### 15.4 Aggregates and slices

A command slice may load one aggregate, invoke behavior, and save changes.

```csharp
var order = await _db.Orders
    .Include(o => o.Lines)
    .SingleAsync(o => o.Id == command.OrderId, cancellationToken);

order.Cancel(command.Reason);

await _db.SaveChangesAsync(cancellationToken);
```

A query slice should not be forced through aggregates if it only needs a projection.

```csharp
var response = await _db.Orders
    .Where(o => o.Id == query.OrderId)
    .Select(o => new OrderDetailsResponse(...))
    .SingleAsync(cancellationToken);
```

### 15.5 Avoid entity-driven slicing

Entity-driven slicing is often too broad:

```text
Features/Orders
Features/Customers
Features/Products
```

This is a start, but it may hide real capabilities. Better:

```text
Ordering/CreateOrder
Ordering/CancelOrder
Catalog/SearchProducts
Catalog/UpdateProductPrice
Fulfillment/ScheduleShipment
Billing/IssueInvoice
```

Business processes often cut across nouns.

---

## 16. Persistence and data access

### 16.1 Direct DbContext inside slice

Acceptable when:

- The dependency is local to the slice.
- ORM is already a chosen application dependency.
- The slice can be tested through integration tests.
- Replacing ORM globally is unlikely or not worth abstracting.

```csharp
public sealed class Handler
{
    private readonly AppDbContext _db;

    public async Task<Response> Handle(Query query, CancellationToken ct)
    {
        return await _db.Orders
            .Where(o => o.Id == query.OrderId)
            .Select(o => new Response(o.Id, o.Status))
            .SingleAsync(ct);
    }
}
```

### 16.2 Repository inside slice

Good when the repository is slice-specific:

```text
Features/Reports/GetMonthlyRevenue/
  Handler.cs
  MonthlyRevenueReadModel.cs
  MonthlyRevenueQuery.cs
```

Avoid global generic repository:

```csharp
IRepository<Order>
```

A generic repository often hides useful query capabilities and creates weak abstractions.

### 16.3 Query object

Useful for reusable complex queries inside a module:

```text
Ordering/
  Queries/
    ActiveCustomerOrdersQuery.cs
  Features/
    GetCustomerOrders/
    ExportCustomerOrders/
```

Only extract when reuse is real and stable.

### 16.4 Multiple data access styles in one app

VSA allows this:

```text
Orders/CreateOrder      # EF Core aggregate
Orders/GetOrderDetails  # Dapper SQL projection
Reports/GetRevenue      # Materialized view
Search/SearchProducts   # Elasticsearch/OpenSearch
```

This is not inconsistency if each slice chooses the tool that fits the job.

### 16.5 Database migrations

Options:

1. Global migration project for whole database.
2. Module-owned migrations.
3. Slice-adjacent migration notes.
4. Migration files grouped by bounded context.

For modular monoliths, module-owned migrations are often clearer:

```text
Modules/Ordering/Infrastructure/Migrations/
Modules/Payments/Infrastructure/Migrations/
```

Rules:

- Do not put schema migration logic inside request handlers.
- Keep migration ownership explicit.
- If a schema change affects many slices, identify whether a shared model is too broad.

---

## 17. Validation strategy

### 17.1 Validation types

| Type | Example | Where it belongs |
|---|---|---|
| Syntactic input validation | Required field, max length, valid enum | Slice validator |
| Authorization validation | User may create order | Endpoint policy or authorization behavior |
| Business invariant | Cannot cancel shipped order | Domain model or command handler |
| Cross-aggregate rule | Customer credit limit | Domain service, application service, policy service |
| External validation | Payment method is valid | Gateway/service inside command slice |
| Persistence constraint | Unique email | Database constraint plus command error mapping |

### 17.2 Slice validator

```csharp
public sealed class Validator : AbstractValidator<CreateOrderCommand>
{
    public Validator()
    {
        RuleFor(x => x.CustomerId).NotEmpty();
        RuleFor(x => x.Lines).NotEmpty();
    }
}
```

### 17.3 Domain invariant

```csharp
public void Cancel(string reason)
{
    if (Status == OrderStatus.Shipped)
        throw new DomainException("Shipped orders cannot be cancelled.");

    Status = OrderStatus.Cancelled;
    CancellationReason = reason;
}
```

### 17.4 Do not duplicate domain invariants in validators

Bad:

```csharp
RuleFor(x => x.OrderStatus).NotEqual(OrderStatus.Shipped);
```

This can go stale and can be bypassed by another caller.

Validator should check request shape. Domain model should enforce business truth.

### 17.5 Validation pipeline

A validation pipeline can run all validators for a request before the handler.

Benefits:

- Removes boilerplate from handlers.
- Ensures consistent error format.
- Keeps validators slice-local.

Risks:

- Hidden execution order.
- Harder debugging if agent forgets behavior registration.
- Business rules accidentally placed in validators.

---

## 18. Authorization strategy

### 18.1 Authorization belongs near the entry point

Endpoint-level:

```csharp
app.MapPost("/orders", Handle)
   .RequireAuthorization("Orders.Create");
```

Slice-level metadata:

```csharp
public sealed record CreateOrderCommand(...) : IRequirePermission
{
    public string Permission => "Orders.Create";
}
```

Pipeline behavior:

```csharp
public sealed class AuthorizationBehavior<TRequest, TResponse>
    : IPipelineBehavior<TRequest, TResponse>
{
    // Check marker interfaces or attributes.
}
```

### 18.2 Feature-specific authorization

If the rule is behavior-specific, keep it in the slice.

Example:

```csharp
if (!await _permissions.CanCancelOrder(userId, order.Id))
    return Errors.Forbidden();
```

### 18.3 Domain authorization versus application authorization

- "User has permission `Orders.Cancel`" is application authorization.
- "Order cannot be cancelled after shipment" is a domain invariant.
- "Only assigned manager can approve this invoice" may involve both application identity and domain state.

---

## 19. Error handling

### 19.1 Prefer explicit result types or consistent exception mapping

Options:

```csharp
Result<T>
OneOf<TSuccess, TError>
TypedResults
ProblemDetails
Exceptions mapped by middleware
```

Pick one style per application or module.

### 19.2 Error placement

| Error type | Where to handle |
|---|---|
| Input validation | Validator plus validation behavior/filter |
| Not found | Handler or query helper |
| Forbidden | Authorization behavior or handler |
| Domain rule violation | Domain exception/result, mapped by behavior |
| Concurrency conflict | Handler or persistence behavior |
| Infrastructure failure | Middleware/logging/retry policy |
| External service failure | Gateway wrapper plus slice-specific decision |

### 19.3 Avoid leaking persistence errors

Bad:

```json
{
  "error": "duplicate key value violates unique constraint users_email_key"
}
```

Better:

```json
{
  "type": "https://example.com/errors/email-already-used",
  "title": "Email already used",
  "status": 409
}
```

---

## 20. Transactions, consistency, and workflows

### 20.1 Transaction per command slice

Most command slices should have one clear transaction boundary.

```text
Validate input
Authorize
Load aggregate
Invoke domain behavior
Save changes
Record outbox messages
Commit
Return response
```

### 20.2 Avoid distributed transactions by default

When one business process spans modules or services, prefer:

- Domain events.
- Integration events.
- Outbox pattern.
- Process manager.
- Saga orchestration.
- Idempotent consumers.
- Compensating actions.

### 20.3 Workflow across slices

Example workflow:

```text
OrderPlaced
  -> ReserveInventory
  -> CapturePayment
  -> ScheduleShipment
  -> SendConfirmationEmail
```

Do not hide this workflow by direct handler calls.

Better structures:

```text
Features/Checkout/PlaceOrder
Features/Inventory/ReserveInventory
Features/Payments/CapturePayment
Features/Fulfillment/ScheduleShipment
Workflows/OrderFulfillmentProcess
```

### 20.4 Event-driven interaction

Events can decouple slices, but do not remove coupling completely. They move coupling to contracts and eventual consistency.

Rules:

- Use past-tense names: `OrderPlaced`, `PaymentCaptured`.
- Events should express business meaning, not database updates.
- Version public events.
- Keep event consumers idempotent.
- Use outbox for reliable publication.
- Avoid publishing events for every internal method call.

---

## 21. Testing strategy

### 21.1 Testing pyramid for VSA

Recommended:

```text
Many slice-level integration tests
Some domain unit tests
Some pure handler tests
Few full end-to-end tests
Architecture boundary tests
Contract tests for public APIs/events
```

### 21.2 Slice integration tests

A slice integration test exercises the real entry point and enough infrastructure to catch wiring errors.

Example:

```csharp
[Fact]
public async Task CreateOrder_returns_created_order_id()
{
    var client = _factory.CreateClient();

    var response = await client.PostAsJsonAsync("/orders", new
    {
        customerId = TestIds.CustomerId,
        lines = new[]
        {
            new { productId = TestIds.ProductId, quantity = 2 }
        }
    });

    response.StatusCode.Should().Be(HttpStatusCode.Created);

    var body = await response.Content.ReadFromJsonAsync<CreateOrderResponse>();
    body!.OrderId.Should().NotBeEmpty();
}
```

This catches:

- Route registration.
- JSON binding.
- Validation behavior.
- Authorization configuration, if enabled.
- Handler registration.
- Persistence mapping.
- Error mapping.

### 21.3 Handler tests

Useful when:

- Handler contains non-trivial branching.
- Integration tests are too slow for edge cases.
- You can use real domain objects and fake gateways.

Avoid testing implementation details that are already covered by integration tests.

### 21.4 Domain unit tests

Test domain invariants directly.

```csharp
[Fact]
public void Shipped_order_cannot_be_cancelled()
{
    var order = OrderTestBuilder.ShippedOrder();

    var act = () => order.Cancel("Customer request");

    act.Should().Throw<DomainException>()
        .WithMessage("Shipped orders cannot be cancelled.");
}
```

### 21.5 Query tests

For complex SQL/projections, use real database integration tests or snapshot tests.

Do not mock ORM LINQ queries heavily. It gives false confidence.

### 21.6 Architecture tests

Use architecture tests to enforce boundaries.

Examples:

- Feature namespaces must not depend on other feature namespaces.
- Modules must not reference another module's internals.
- Public contracts must not depend on infrastructure.
- Domain must not depend on web framework.
- Handlers must not call other handlers directly.

Pseudo-example:

```csharp
[Fact]
public void Ordering_features_should_not_reference_payments_internals()
{
    Types.InAssembly(OrderingAssembly)
        .That()
        .ResideInNamespace("App.Modules.Ordering.Features")
        .Should()
        .NotHaveDependencyOn("App.Modules.Payments.Features")
        .GetResult()
        .IsSuccessful
        .Should()
        .BeTrue();
}
```

### 21.7 Contract tests

Use for:

- Public REST APIs.
- GraphQL schema.
- gRPC contracts.
- Integration events.
- External message contracts.

### 21.8 Test folder placement

Option A, tests mirror production:

```text
tests/
  App.Tests/
    Features/
      Orders/
        CreateOrderTests.cs
```

Option B, tests near slices:

```text
src/
  Features/
    Orders/
      CreateOrder/
        Handler.cs
        Tests.cs
```

Option C, hybrid:

```text
Features/Orders/CreateOrder/
tests/Features/Orders/CreateOrder/
```

Choose based on language and build conventions.

---

## 22. Observability

Each slice should be observable as a behavior.

### 22.1 Logging

Log at behavior boundaries:

```text
CreateOrder started
CreateOrder validation failed
CreateOrder completed
CreateOrder failed
```

Include:

- Correlation ID
- Tenant ID, if applicable
- User ID, if safe
- Slice name
- Request ID
- Duration
- Domain IDs, if safe

Avoid logging sensitive payloads.

### 22.2 Metrics

Common metrics:

```text
slice_requests_total{slice="CreateOrder", outcome="success"}
slice_duration_seconds{slice="CreateOrder"}
slice_validation_failures_total{slice="CreateOrder"}
slice_errors_total{slice="CreateOrder", error_type="concurrency"}
```

### 22.3 Tracing

Create spans around:

- Endpoint.
- Handler.
- Database query.
- External API call.
- Event publication.
- Message consumption.

### 22.4 Naming convention

Use stable slice names:

```text
Ordering.CreateOrder
Ordering.CancelOrder
Payments.CapturePayment
Reports.GetMonthlyRevenue
```

---

## 23. Scalability considerations

### 23.1 Codebase scalability

VSA scales a codebase by reducing change scatter.

Benefits:

- Easier feature navigation.
- Smaller pull requests.
- Less accidental shared-code modification.
- Easier ownership.
- Faster onboarding by use case.

Risks:

- Too many tiny folders.
- Duplicate logic without governance.
- Inconsistent patterns across slices.
- Hidden workflows.
- Weak module boundaries.

### 23.2 Team scalability

Ownership models:

1. Team owns module:
   ```text
   Team Ordering owns Modules/Ordering
   Team Payments owns Modules/Payments
   ```

2. Team owns feature area:
   ```text
   Team Checkout owns Checkout and Payment initiation slices
   ```

3. Platform team owns shared infrastructure:
   ```text
   Shared/Auth
   Shared/Observability
   Shared/Persistence
   Shared/Messaging
   ```

VSA works best when ownership follows business capability, not technical layer.

### 23.3 Runtime scalability

VSA itself does not improve runtime performance automatically. It helps agents choose performance strategies per slice:

- Read-heavy query: cache, projection, read replica, materialized view.
- Write-heavy command: optimistic concurrency, queue, batch, outbox.
- Expensive report: async job, export file, precomputed table.
- External API: retry, circuit breaker, timeout, idempotency.
- Search: specialized search index.

### 23.4 Deployment scalability

VSA can be a stepping stone toward modular monoliths or services.

Good extraction candidate:

- Module has high cohesion.
- Module has low coupling to others.
- Module owns its data or has clear data contracts.
- Module has explicit public API/events.
- Module has tests and observability.
- Module has independent scaling or deployment need.

Bad extraction candidate:

- Slices still call each other's internals.
- Shared database tables are heavily coupled.
- Public contracts are not stable.
- Workflows are implicit.
- Tests require entire monolith state.

---

## 24. Relationship to other architectures

### 24.1 VSA versus layered architecture

| Concern | Layered architecture | Vertical Slice Architecture |
|---|---|---|
| Primary organization | Technical role | Feature/use case/capability |
| Change locality | Often scattered | Usually localized |
| Abstraction style | Global layer abstractions | Per-slice decisions |
| Common failure | Service/repository graveyards | Too many slices or duplication |
| Best use | Stable technical separation | Feature delivery and behavior clarity |

### 24.2 VSA versus Clean Architecture

Clean Architecture focuses on dependency direction and protecting business rules from frameworks.

VSA focuses on cohesion around use cases and reducing feature change scatter.

They are not mutually exclusive.

A slice can still respect dependency rules:

```text
CreateOrder/
  Endpoint.cs       # outer web detail
  Command.cs        # application input
  Handler.cs        # use case
Ordering/Domain/
  Order.cs          # domain
Infrastructure/
  AppDbContext.cs   # persistence detail
```

But strict Clean Architecture with many projects can fight VSA locality if every slice must be split across layers.

### 24.3 VSA versus Hexagonal Architecture

Hexagonal Architecture focuses on ports and adapters.

VSA can use ports where a slice needs them:

```text
CapturePayment/
  Handler.cs
  IPaymentGateway.cs
  StripePaymentGateway.cs
```

But do not create ports for every dependency by default. Create them when they protect meaningful volatility or enable testing without hiding important behavior.

### 24.4 VSA versus modular monolith

A modular monolith defines coarse business boundaries inside one deployable app. VSA defines how behavior is organized inside those boundaries.

They work well together:

```text
Modules/Ordering/Features/CreateOrder
Modules/Payments/Features/CapturePayment
Modules/Shipping/Features/ScheduleShipment
```

### 24.5 VSA versus microservices

Microservices are deployment and ownership boundaries. VSA is code and capability organization.

A microservice can use VSA internally.

A monolith can use VSA without becoming microservices.

Do not split services just because slices exist.

### 24.6 VSA versus Feature-Sliced Design frontend

Feature-Sliced Design in frontend ecosystems has similar goals: organize by product capability and isolate feature logic. VSA can align backend and frontend, but frontend ecosystems often add layers like `app`, `pages`, `widgets`, `features`, `entities`, and `shared`.

Use the shared principle, not necessarily the same folder names.

---

## 25. Common patterns

### Pattern: Request-EndPoint-Response

Use for API slices.

```text
CreateProduct/
  Request.cs
  Endpoint.cs
  Response.cs
```

Good when the endpoint itself is the feature boundary.

### Pattern: Command-Handler-Validator

```text
CreateOrder/
  Command.cs
  Handler.cs
  Validator.cs
  Endpoint.cs
```

Good when using CQRS or mediator.

### Pattern: Query-Projection-Response

```text
GetOrderDetails/
  Query.cs
  Projection.cs
  Handler.cs
  Response.cs
```

Good for read models.

### Pattern: Slice-private repository

```text
GetRevenueReport/
  Handler.cs
  RevenueReportSql.cs
```

Good for complex query logic that should not become global.

### Pattern: Domain model plus slices

```text
Ordering/
  Domain/
    Order.cs
  Features/
    CreateOrder/
    CancelOrder/
    SubmitOrder/
```

Good when multiple command slices enforce the same invariants.

### Pattern: Public contracts folder

```text
Ordering/
  Contracts/
    OrderPlaced.cs
    OrderCancelled.cs
```

Good for event-driven integration.

### Pattern: Workflow/process manager

```text
Workflows/
  OrderFulfillment/
    OrderFulfillmentProcess.cs
```

Good when a business process coordinates multiple capabilities.

### Pattern: Shared kernel

```text
SharedKernel/
  Money.cs
  EmailAddress.cs
  TenantId.cs
```

Good for stable primitives.

### Pattern: Architecture tests

```text
ArchitectureTests/
  FeatureDependencyTests.cs
  ModuleBoundaryTests.cs
```

Good for preventing erosion.

---

## 26. Anti-patterns

### Anti-pattern: Layer folders inside every slice

Bad:

```text
CreateOrder/
  Controllers/
  Services/
  Repositories/
  DTOs/
```

This duplicates layered architecture many times.

Better:

```text
CreateOrder/
  Endpoint.cs
  Command.cs
  Handler.cs
  Validator.cs
  Response.cs
```

### Anti-pattern: Service graveyard

Bad:

```text
Services/
  OrderService.cs
  UserService.cs
  PaymentService.cs
  CommonService.cs
  HelperService.cs
```

Symptoms:

- Services contain unrelated methods.
- Method names are verbs but class names are nouns.
- Many features modify the same service.
- Services depend on many repositories and other services.

Better:

```text
Features/Orders/CreateOrder
Features/Orders/CancelOrder
Features/Payments/CapturePayment
```

### Anti-pattern: Shared DTO everywhere

Bad:

```csharp
public sealed class OrderDto
{
    public Guid Id { get; set; }
    public string Status { get; set; }
    public decimal Total { get; set; }
    public List<OrderLineDto> Lines { get; set; }
    public string InternalRiskFlag { get; set; }
}
```

Used by:

```text
GetOrderDetails
ListOrders
ExportOrders
CancelOrder
AdminOrderPage
```

Problems:

- Overfetching.
- Accidental data exposure.
- Hard versioning.
- Changes break unrelated endpoints.

Better: response per slice.

### Anti-pattern: Slice calls another slice's handler

Bad:

```csharp
await _mediator.Send(new CancelOrderCommand(orderId));
```

from inside another handler, unless you are intentionally using a public command bus and have designed the coupling. Usually this creates hidden workflows.

Better:

- Extract shared domain method.
- Publish event.
- Introduce workflow orchestrator.
- Make explicit application service.

### Anti-pattern: Everything in one giant handler

Bad:

```text
CreateOrder/Handler.cs  # 900 lines
```

Symptoms:

- Validation, persistence, external calls, mapping, and domain rules all mixed.
- Hard to test.
- Hard to read.
- Many private methods with unclear order.

Better:

- Extract domain model.
- Extract policy class.
- Extract gateway.
- Extract mapper only if mapping is non-trivial.
- Split workflow if it has independent steps.

### Anti-pattern: Premature generic abstractions

Bad:

```csharp
IRepository<T>
IService<T>
IValidatorService<T>
IMapperService<TSource, TDestination>
```

These often hide behavior and force all slices into one shape.

Better:

- Direct dependency in the slice.
- Slice-specific query.
- Domain-specific interface.

### Anti-pattern: "No sharing ever"

This is dogma. Sharing is allowed when intentional.

Good sharing:

- Cross-cutting infrastructure.
- Stable domain primitives.
- Public contracts.
- Common observability.
- Security policies.
- Shared domain model inside a module.

Bad sharing:

- Random helpers.
- Shared DTOs for convenience.
- One global service with unrelated methods.
- Feature internals used by other features.

### Anti-pattern: "VSA means no architecture"

VSA still requires architecture decisions:

- Module boundaries.
- Data ownership.
- Workflow design.
- Public contracts.
- Dependency rules.
- Test strategy.
- Observability.
- Error handling.
- Deployment strategy.

### Anti-pattern: Entity-first organization only

Bad:

```text
Features/Order
Features/Customer
Features/Product
```

This can still become a noun-oriented mini-layer.

Better:

```text
Ordering/CreateOrder
Ordering/CancelOrder
CustomerManagement/RegisterCustomer
Catalog/SearchProducts
Pricing/ChangeProductPrice
```

### Anti-pattern: Framework-first folder names

Bad:

```text
Controllers/
MediatR/
FluentValidation/
EntityFramework/
```

Better:

```text
Orders/CreateOrder/
Payments/CapturePayment/
Reports/GetRevenue/
```

Architecture should communicate the business, not the framework.

---

## 27. Edge cases and recommended handling

### Edge case: Two slices need the same validation

Ask whether it is input shape validation or domain rule.

- Input shape: duplicate or extract a small validator extension.
- Domain rule: move to domain model.
- Authorization rule: move to policy.
- External check: extract domain/application service.

### Edge case: Two slices need the same query

If query is simple, duplicate.

If query is complex and stable, extract module-level query object:

```text
Ordering/ReadModels/CustomerOrderSummaryQuery.cs
```

Do not put it inside one slice and import it from another.

### Edge case: A slice needs data from another module

Prefer a public contract or API.

Options:

- Read from a published read model.
- Call explicit module API.
- Consume integration event and store local projection.
- Use direct database read only if architecture permits and ownership is clear.

Avoid reaching into another module's tables casually.

### Edge case: Transaction spans multiple capabilities

Avoid direct cross-slice calls in one transaction. Consider:

- Redesign aggregate boundary.
- Use eventual consistency.
- Use saga/process manager.
- Use outbox.
- Use compensating actions.
- If single database transaction is truly required, create an explicit application workflow and document the coupling.

### Edge case: A feature has multiple endpoints

Group under one feature folder if they represent one coherent capability.

```text
Checkout/
  StartCheckout/
  ApplyCoupon/
  ConfirmCheckout/
```

Or:

```text
Checkout/
  Start.cs
  ApplyCoupon.cs
  Confirm.cs
```

Avoid one `CheckoutHandler` with many operations.

### Edge case: A slice becomes huge

Refactoring ladder:

1. Extract private methods.
2. Extract value objects.
3. Extract domain entity/aggregate behavior.
4. Extract policy class.
5. Extract gateway for external dependency.
6. Split command and query.
7. Split workflow step.
8. Promote to module if capability is large.

### Edge case: Need reusable UI components

Move truly generic UI to `shared/ui`.

Keep feature-specific components local.

```text
shared/ui/Button.tsx
features/orders/order-details/OrderStatusBadge.tsx
```

Move `OrderStatusBadge` only if multiple features use it and it represents stable shared language.

### Edge case: API versioning

Options:

```text
Orders/GetOrder/v1/
Orders/GetOrder/v2/
```

or

```text
Orders/GetOrder/
  V1/
  V2/
```

Use separate response models per version. Do not make one response type with many optional fields unless unavoidable.

### Edge case: Multi-tenancy

Common tenant resolution should be cross-cutting.

Tenant-specific rules may be slice-specific.

```text
Shared/Tenancy/
Features/Orders/CreateOrder/
```

Rules:

- Tenant ID must be part of authorization and data filters.
- Tests must cover cross-tenant isolation.
- Avoid passing tenant implicitly if security needs explicit checks.

### Edge case: Idempotency

For command slices called by unreliable clients or message delivery:

```text
CreatePayment/
  Command.cs
  Handler.cs
  IdempotencyPolicy.cs
```

Generic behavior can enforce idempotency for requests implementing a marker:

```csharp
public interface IIdempotentCommand
{
    string IdempotencyKey { get; }
}
```

### Edge case: Eventual consistency in queries

If command writes and query reads a projection, the query may lag.

Make the API contract explicit:

- Return `202 Accepted` for async operations.
- Return operation status endpoint.
- Use read-your-writes cache when needed.
- Update projection synchronously only if required.

### Edge case: Bulk operations

Avoid one generic bulk endpoint that bypasses all rules.

Design bulk as a slice:

```text
Orders/BulkCancelOrders/
  Command.cs
  Handler.cs
  Validator.cs
  Result.cs
```

Decide whether partial failure is allowed.

### Edge case: External integrations

Keep integration-specific code either:

- Slice-local if only one slice uses it.
- In an infrastructure adapter if reused.
- Behind a domain-specific gateway interface if behavior needs protection.

Example:

```text
Payments/CapturePayment/
  Handler.cs
  IPaymentGateway.cs
Infrastructure/Stripe/
  StripePaymentGateway.cs
```

### Edge case: Reports and analytics

Reports are excellent query slices.

```text
Reports/GetMonthlyRevenue/
  Query.cs
  Sql.cs
  Handler.cs
  Response.cs
```

Do not force reports through domain aggregates.

---

## 28. Migration playbook from layered architecture

### Step 1: Pick one feature

Start with a narrow behavior:

```text
CreateOrder
GetOrderDetails
CancelOrder
```

Avoid migrating the whole architecture at once.

### Step 2: Identify files touched by that feature

Example:

```text
OrdersController.Create
OrderService.Create
OrderRepository.Add
CreateOrderDto
OrderDto
CreateOrderValidator
OrderMappingProfile
```

### Step 3: Create a slice folder

```text
Features/Orders/CreateOrder/
```

Move or copy feature-specific files into it.

### Step 4: Make request and response feature-specific

Do not reuse giant shared DTOs unless required temporarily.

### Step 5: Inline unnecessary abstractions

If `OrderService.Create` only serves `CreateOrder`, move the method into the handler.

If `OrderRepository.Add` wraps one ORM call, consider direct persistence in the slice.

### Step 6: Keep real domain model where appropriate

If `Order` enforces invariants, keep it in domain/module.

If it is just a data record used by one slice, keep it local or persistence-specific.

### Step 7: Add slice integration tests

Test behavior before and after migration.

### Step 8: Add architecture tests

Prevent new code from importing slice internals.

### Step 9: Repeat incrementally

Migrate feature by feature.

### Step 10: Delete dead layer artifacts

Remove empty services, repositories, DTOs, mapping profiles, and validators as slices absorb them.

---

## 29. Code review checklist for agents

When reviewing a VSA change, check:

### Boundary

- [ ] Is this organized by behavior/capability rather than technical role?
- [ ] Does the slice contain the files that change together?
- [ ] Does the slice avoid importing another slice's internals?
- [ ] Are public contracts clearly separated from private implementation?
- [ ] Is shared code intentionally placed?

### Naming

- [ ] Is the slice named as a business action or use case?
- [ ] Are command names imperative?
- [ ] Are event names past tense?
- [ ] Are query names descriptive of the read model?
- [ ] Does the folder structure reveal what the system does?

### CQRS

- [ ] Does the command mutate state?
- [ ] Does the query avoid side effects?
- [ ] Are read and write models separated where useful?
- [ ] Is separate storage avoided unless needed?

### Validation and authorization

- [ ] Is input validation in the slice?
- [ ] Are domain invariants enforced in the domain model or command logic?
- [ ] Is authorization close to the entry point or a clear behavior?
- [ ] Are validation errors mapped consistently?

### Persistence

- [ ] Is direct data access acceptable for this slice?
- [ ] Is a repository or abstraction justified?
- [ ] Are queries optimized for response shape?
- [ ] Are transactions clear?

### Testing

- [ ] Is there at least one slice-level test?
- [ ] Are domain invariants unit-tested?
- [ ] Are boundary rules architecture-tested?
- [ ] Are contracts tested when public?

### Observability

- [ ] Can logs identify the slice?
- [ ] Are metrics tagged by slice?
- [ ] Are external calls traced?
- [ ] Is sensitive data protected?

### Maintainability

- [ ] Is the handler small enough to understand?
- [ ] Is duplication acceptable or should it be extracted?
- [ ] Has the agent avoided generic abstractions?
- [ ] Is the design easy to change for the likely next requirement?

---

## 30. Implementation checklist for a new slice

Use this checklist when creating a slice.

1. Name the behavior.
   - Good: `CreateOrder`, `CancelOrder`, `GetOrderDetails`.
   - Bad: `OrderService`, `ManageOrders`.

2. Choose slice location.
   - Small app: `Features/Orders/CreateOrder`.
   - Modular app: `Modules/Ordering/Features/CreateOrder`.

3. Define input contract.
   - Command, query, request DTO, event payload, or form model.

4. Define output contract.
   - Response DTO, result type, event, status, or file.

5. Decide entry point.
   - HTTP endpoint, message consumer, job, CLI, UI action.

6. Add validation.
   - Keep syntactic validation in the slice.

7. Add authorization.
   - Endpoint policy, behavior, or explicit check.

8. Implement behavior.
   - Use transaction script, domain model, aggregate, or service as needed.

9. Handle persistence.
   - Direct ORM, query, repository, or adapter based on need.

10. Handle errors.
    - Not found, validation, forbidden, conflict, domain violation.

11. Add tests.
    - Integration test for full slice.
    - Unit tests for domain rules.
    - Edge cases.

12. Register slice.
    - Route map, DI registration, module config, message handler registration.

13. Add observability.
    - Logging, metrics, trace span.

14. Update documentation.
    - API examples, event contracts, module README, if relevant.

15. Run architecture checks.
    - Ensure no forbidden dependencies.

---

## 31. Agent prompts and expected outputs

### Prompt: Design architecture

User:

> Design a folder structure for an order management API using Vertical Slice Architecture.

Agent output should include:

- Feature/module breakdown.
- Example folder tree.
- Dependency rules.
- CQRS choice.
- Testing layout.
- Shared code policy.
- Tradeoffs.

### Prompt: Refactor layered code

User:

> Refactor this controller/service/repository into Vertical Slice Architecture.

Agent workflow:

1. Identify use cases in the controller.
2. Create one slice per use case.
3. Move request/response models local.
4. Inline thin service methods.
5. Keep domain rules in domain model.
6. Replace shared DTOs with slice responses.
7. Add tests.
8. Remove unused layered files.

### Prompt: Review design

User:

> Is this Vertical Slice Architecture correct?

Agent should check:

- Is it feature-first?
- Are slice boundaries respected?
- Is shared logic deliberate?
- Is it just layers inside feature folders?
- Are commands and queries clear?
- Are tests aligned with slices?
- Are dependencies one-way and explicit?

### Prompt: Add CQRS

User:

> Add CQRS to this VSA project.

Agent should:

- Split state-changing slices into commands.
- Split read-only slices into queries.
- Avoid separate databases unless needed.
- Add mediator only if useful.
- Add validation and transaction behaviors if beneficial.
- Keep command/query files inside feature folders.

---

## 32. Real-world example: order management API

### Folder tree

```text
src/
  Modules/
    Ordering/
      Domain/
        Order.cs
        OrderLine.cs
        OrderStatus.cs
      Features/
        CreateOrder/
          Endpoint.cs
          Command.cs
          Handler.cs
          Validator.cs
          Response.cs
        CancelOrder/
          Endpoint.cs
          Command.cs
          Handler.cs
          Validator.cs
        GetOrderDetails/
          Endpoint.cs
          Query.cs
          Handler.cs
          Response.cs
        ListCustomerOrders/
          Endpoint.cs
          Query.cs
          Handler.cs
          Response.cs
      Infrastructure/
        OrderingDbContext.cs
        OrderingMappings.cs
      Contracts/
        OrderPlaced.cs
        OrderCancelled.cs
    Payments/
      Features/
        CapturePayment/
        RefundPayment/
      Contracts/
        PaymentCaptured.cs
    Shipping/
      Features/
        ScheduleShipment/
      Contracts/
        ShipmentScheduled.cs
  Shared/
    Auth/
    Errors/
    Messaging/
    Observability/
    Time/
  Host/
    Program.cs
tests/
  Modules/
    Ordering/
      CreateOrderTests.cs
      CancelOrderTests.cs
      GetOrderDetailsTests.cs
  Architecture/
    ModuleBoundaryTests.cs
```

### CreateOrder command

```csharp
namespace App.Modules.Ordering.Features.CreateOrder;

public sealed record Command(
    Guid CustomerId,
    IReadOnlyList<Line> Lines
) : IRequest<Response>;

public sealed record Line(Guid ProductId, int Quantity);

public sealed record Response(Guid OrderId);
```

### Validator

```csharp
public sealed class Validator : AbstractValidator<Command>
{
    public Validator()
    {
        RuleFor(x => x.CustomerId).NotEmpty();
        RuleFor(x => x.Lines).NotEmpty();

        RuleForEach(x => x.Lines).ChildRules(line =>
        {
            line.RuleFor(x => x.ProductId).NotEmpty();
            line.RuleFor(x => x.Quantity).GreaterThan(0);
        });
    }
}
```

### Handler

```csharp
public sealed class Handler : IRequestHandler<Command, Response>
{
    private readonly OrderingDbContext _db;
    private readonly IEventBus _eventBus;

    public Handler(OrderingDbContext db, IEventBus eventBus)
    {
        _db = db;
        _eventBus = eventBus;
    }

    public async Task<Response> Handle(Command command, CancellationToken ct)
    {
        var order = Order.Create(command.CustomerId);

        foreach (var line in command.Lines)
        {
            order.AddLine(line.ProductId, line.Quantity);
        }

        _db.Orders.Add(order);

        await _db.SaveChangesAsync(ct);

        await _eventBus.Publish(new OrderPlaced(order.Id, order.CustomerId), ct);

        return new Response(order.Id);
    }
}
```

### Endpoint

```csharp
public static class Endpoint
{
    public static void Map(IEndpointRouteBuilder app)
    {
        app.MapPost("/orders", async (
            Command command,
            ISender sender,
            CancellationToken ct) =>
        {
            var response = await sender.Send(command, ct);
            return Results.Created($"/orders/{response.OrderId}", response);
        })
        .WithTags("Orders")
        .RequireAuthorization("Orders.Create");
    }
}
```

### Query slice with optimized projection

```csharp
namespace App.Modules.Ordering.Features.GetOrderDetails;

public sealed record Query(Guid OrderId) : IRequest<Response>;

public sealed record Response(
    Guid Id,
    string Status,
    decimal Total,
    IReadOnlyList<LineResponse> Lines);

public sealed record LineResponse(
    Guid ProductId,
    string ProductName,
    int Quantity,
    decimal UnitPrice);

public sealed class Handler : IRequestHandler<Query, Response>
{
    private readonly OrderingDbContext _db;

    public Handler(OrderingDbContext db)
    {
        _db = db;
    }

    public async Task<Response> Handle(Query query, CancellationToken ct)
    {
        var response = await _db.Orders
            .Where(o => o.Id == query.OrderId)
            .Select(o => new Response(
                o.Id,
                o.Status.ToString(),
                o.Lines.Sum(l => l.Quantity * l.UnitPrice),
                o.Lines.Select(l => new LineResponse(
                    l.ProductId,
                    l.ProductName,
                    l.Quantity,
                    l.UnitPrice)).ToList()))
            .SingleOrDefaultAsync(ct);

        if (response is null)
        {
            throw new NotFoundException("Order not found.");
        }

        return response;
    }
}
```

Notice that the query does not load the aggregate to enforce behavior. It shapes the read model directly.

---

## 33. Real-world example: TypeScript Fastify VSA

### Folder tree

```text
src/
  features/
    orders/
      create-order/
        route.ts
        schema.ts
        handler.ts
        types.ts
        test.ts
      get-order/
        route.ts
        handler.ts
        response.ts
    payments/
      capture-payment/
  shared/
    db.ts
    auth.ts
    errors.ts
    observability.ts
```

### Schema

```ts
// features/orders/create-order/schema.ts
import { z } from "zod";

export const createOrderBodySchema = z.object({
  customerId: z.string().uuid(),
  lines: z.array(z.object({
    productId: z.string().uuid(),
    quantity: z.number().int().positive(),
  })).min(1),
});

export type CreateOrderBody = z.infer<typeof createOrderBodySchema>;
```

### Handler

```ts
// features/orders/create-order/handler.ts
import { db } from "../../../shared/db";
import { CreateOrderBody } from "./schema";

export async function createOrder(body: CreateOrderBody) {
  return db.$transaction(async (tx) => {
    const order = await tx.order.create({
      data: { customerId: body.customerId },
    });

    await tx.orderLine.createMany({
      data: body.lines.map((line) => ({
        orderId: order.id,
        productId: line.productId,
        quantity: line.quantity,
      })),
    });

    return { orderId: order.id };
  });
}
```

### Route

```ts
// features/orders/create-order/route.ts
import { FastifyInstance } from "fastify";
import { createOrderBodySchema } from "./schema";
import { createOrder } from "./handler";

export async function createOrderRoute(app: FastifyInstance) {
  app.post("/orders", async (request, reply) => {
    const parseResult = createOrderBodySchema.safeParse(request.body);

    if (!parseResult.success) {
      return reply.code(400).send({
        error: "Validation failed",
        details: parseResult.error.flatten(),
      });
    }

    const response = await createOrder(parseResult.data);
    return reply.code(201).send(response);
  });
}
```

---

## 34. Real-world example: Java Spring Boot VSA

### Folder tree

```text
src/main/java/com/example/app/
  ordering/
    domain/
      Order.java
      OrderLine.java
    features/
      createorder/
        CreateOrderController.java
        CreateOrderRequest.java
        CreateOrderHandler.java
        CreateOrderResponse.java
      getorderdetails/
        GetOrderDetailsController.java
        GetOrderDetailsHandler.java
        GetOrderDetailsResponse.java
    infrastructure/
      OrderingJpaRepository.java
```

### Request and response

```java
public record CreateOrderRequest(
    UUID customerId,
    List<CreateOrderLineRequest> lines
) {}

public record CreateOrderLineRequest(
    UUID productId,
    int quantity
) {}

public record CreateOrderResponse(UUID orderId) {}
```

### Controller

```java
@RestController
@RequestMapping("/orders")
class CreateOrderController {
    private final CreateOrderHandler handler;

    CreateOrderController(CreateOrderHandler handler) {
        this.handler = handler;
    }

    @PostMapping
    ResponseEntity<CreateOrderResponse> create(@RequestBody CreateOrderRequest request) {
        CreateOrderResponse response = handler.handle(request);
        return ResponseEntity.created(URI.create("/orders/" + response.orderId()))
            .body(response);
    }
}
```

### Handler

```java
@Service
class CreateOrderHandler {
    private final OrderRepository repository;

    CreateOrderHandler(OrderRepository repository) {
        this.repository = repository;
    }

    CreateOrderResponse handle(CreateOrderRequest request) {
        Order order = Order.create(request.customerId());

        for (CreateOrderLineRequest line : request.lines()) {
            order.addLine(line.productId(), line.quantity());
        }

        repository.save(order);

        return new CreateOrderResponse(order.id());
    }
}
```

---

## 35. Documentation strategy

Each module or feature area can have a README:

```text
Modules/Ordering/README.md
```

Recommended content:

```markdown
# Ordering module

## Responsibilities

- Create orders
- Cancel orders
- Submit orders
- Query order history

## Public contracts

- OrderPlaced
- OrderCancelled

## Internal slices

- CreateOrder
- CancelOrder
- GetOrderDetails
- ListCustomerOrders

## Data ownership

Owns:
- orders
- order_lines

Does not own:
- payment_transactions
- shipments

## Boundary rules

- Other modules may consume Ordering.Contracts.
- Other modules may not reference Ordering.Features.
- Payment and shipping workflows react to Ordering events.

## Consistency model

- Order creation is synchronous.
- Payment and shipping are eventually consistent.
```

---

## 36. Architecture decision record template

Use this ADR when adopting VSA.

```markdown
# ADR: Adopt Vertical Slice Architecture

## Status

Accepted

## Context

The application is currently organized by technical layers. Feature changes require edits across controllers, services, repositories, validators, DTOs, mapping profiles, and tests. This creates high change scatter and makes feature behavior difficult to understand end to end.

## Decision

We will organize application behavior by vertical slices. A slice represents a business capability, use case, command, query, endpoint, message consumer, or job. Slice-specific request/response models, validators, handlers, mappings, and tests will be placed together.

## Consequences

Positive:
- Feature changes are more localized.
- Slices are easier to test through behavior.
- DTOs and validation become use-case-specific.
- Query and command models can evolve independently.
- Teams can own business capabilities.

Negative:
- Some duplication is expected.
- Agents must avoid creating too many tiny slices.
- Shared logic requires disciplined extraction.
- Architecture tests are needed to prevent boundary erosion.

## Rules

- A slice must not use another slice's internal code.
- Shared code must be stable, intentional, and named by business meaning.
- Domain invariants belong in domain objects or policies, not only in validators.
- Command and query models should be separate when their needs differ.
- Cross-slice workflows should use events, explicit module APIs, or process managers.
```

---

## 37. Practical decision tables

### 37.1 Should this code be shared?

| Question | If yes | If no |
|---|---|---|
| Does it represent a domain concept used by multiple behaviors? | Move to domain/module | Keep in slice |
| Is it generic infrastructure? | Move to shared infrastructure | Keep in slice |
| Is it just convenience reuse? | Prefer duplication | Keep in slice |
| Would changing it require retesting many unrelated features? | Do not share yet | Share if stable |
| Does it have a clear name in business language? | Candidate for domain/shared | Avoid vague helper |

### 37.2 Should I use a repository?

| Situation | Recommendation |
|---|---|
| Simple query projection | Direct query in slice |
| Complex reusable query | Module-level query object |
| Aggregate persistence with invariants | Repository may be useful |
| External persistence technology changes frequently | Consider port/adapter |
| Repository just wraps ORM CRUD | Avoid or keep local |
| Need to hide infrastructure from domain | Repository interface in domain/application, implementation in infra |

### 37.3 Should I use MediatR?

| Situation | Recommendation |
|---|---|
| Need pipeline behaviors | MediatR can help |
| Need command/query dispatch | MediatR can help |
| Tiny API with simple endpoints | Direct handlers may be simpler |
| Team dislikes magic registration | Avoid or document heavily |
| Need message durability | Consider messaging framework or Wolverine, not only MediatR |
| Handlers call other handlers | Revisit design |

### 37.4 Should I split a feature?

| Signal | Action |
|---|---|
| Handler has many unrelated branches | Split into slices |
| Different authorization rules | Split |
| Different transaction boundaries | Split |
| Different read/write model | Split command and query |
| Same user action with one invariant | Keep together |
| Always changes together | Keep together |
| Different team ownership | Split by capability/module |

---

## 38. Common mistakes by AI agents

### Mistake 1: Creating a perfect-looking folder tree but no boundary rules

A folder tree is not architecture. Always specify dependency rules.

### Mistake 2: Making CQRS mandatory

CQRS often fits VSA, but not every simple feature needs mediator, command bus, event sourcing, and projections.

### Mistake 3: Moving files without changing coupling

If the controller still calls `OrderService`, which calls `OrderRepository`, and all slices share the same DTOs, the architecture did not really change.

### Mistake 4: Extracting every duplicate immediately

Two similar lines in two slices are not a problem. Premature sharing creates coupling.

### Mistake 5: Hiding business logic in validators

Validators check input shape. Domain invariants belong in domain behavior or command logic.

### Mistake 6: Treating VSA as anti-Clean Architecture

VSA and Clean Architecture solve different problems. They can coexist.

### Mistake 7: Calling feature handlers from other feature handlers

This creates implicit workflows and hidden coupling.

### Mistake 8: Creating huge "Shared" folder

A `Shared` folder should be small and intentional. If everything is shared, nothing is sliced.

### Mistake 9: Using generic repository everywhere

Generic repositories often create a weak abstraction and block query optimization.

### Mistake 10: Slice names are nouns instead of behaviors

Prefer `CreateOrder` over `Order`.

---

## 39. Quality rubric

Rate a VSA implementation from 1 to 5.

### 1 - Layered architecture with feature labels

- Feature folders exist but contain controllers/services/repositories sublayers.
- Shared DTOs everywhere.
- Slices call each other.
- No boundary tests.

### 2 - Partial feature grouping

- Some request/response/validator files are local.
- Still depends heavily on global services.
- Some shared service classes are large.
- Tests still organized by layer.

### 3 - Functional VSA

- Most behavior is grouped by slice.
- Commands and queries are clear.
- Slice tests exist.
- Shared code is mostly intentional.
- Some boundary leaks remain.

### 4 - Strong VSA

- Slices are cohesive and independently understandable.
- Modules or capabilities are explicit.
- Cross-slice communication uses contracts/events/workflows.
- Architecture tests prevent internal dependencies.
- Query/write models are shaped per use case.

### 5 - Mature capability architecture

- Business capabilities drive structure.
- Modules have clear ownership and contracts.
- Data ownership is explicit.
- Workflows are observable and documented.
- Tests, metrics, traces, and deployment boundaries align with capabilities.
- Extraction to services is possible without major redesign, if ever needed.

---

## 40. Minimal VSA template for agents

Use this as a starting template.

```text
src/
  Features/
    <Area>/
      <UseCase>/
        Endpoint.<ext>
        RequestOrCommand.<ext>
        Handler.<ext>
        Validator.<ext>
        Response.<ext>
        Tests.<ext>
  Shared/
    Auth/
    Errors/
    Observability/
    Persistence/
    Time/
tests/
  Features/
    <Area>/
      <UseCase>Tests.<ext>
```

For larger systems:

```text
src/
  Modules/
    <Capability>/
      Domain/
      Features/
        <UseCase>/
      Infrastructure/
      Contracts/
  SharedKernel/
  Host/
tests/
  Modules/
  Architecture/
```

---

## 41. Final agent rules

When implementing Vertical Slice Architecture, follow these rules:

1. Start from the business behavior, not the framework.
2. Name slices as use cases, commands, queries, or workflows.
3. Keep request, response, validation, handler, endpoint, and tests close.
4. Keep feature-specific DTOs private to the slice.
5. Use domain models for invariants, not just for data.
6. Allow each slice to choose the simplest persistence style.
7. Share only stable, intentional concepts.
8. Do not call another slice's internals.
9. Use events, workflows, public contracts, or domain extraction for cross-slice collaboration.
10. Use CQRS when it clarifies reads versus writes, not as ceremony.
11. Use mediator/pipeline behaviors when they reduce boilerplate and clarify policies.
12. Keep cross-cutting concerns out of handlers when they are truly cross-cutting.
13. Prefer slice integration tests over excessive mocking.
14. Add architecture tests as the codebase grows.
15. Review VSA by cohesion, coupling, and change locality, not by folder aesthetics.

---

## 42. Short answer template for agents

When the user asks, "Where should this go in Vertical Slice Architecture?", answer using this structure:

```markdown
Put it in: `<path>`

Reason:
- It changes with `<slice/use case>`.
- It is not stable shared infrastructure.
- It is not a domain invariant reused across multiple behaviors.

Suggested structure:

```text
Features/<Area>/<UseCase>/
  Endpoint.cs
  Command.cs
  Handler.cs
  Validator.cs
  Response.cs
```

Boundary note:
- Do not import this from another slice.
- If another slice needs the same rule, extract it to `<domain/shared/module location>` only after confirming it is a real shared concept.
```

---

## 43. Review examples

### Example A: Bad layered PR

```text
Controllers/OrdersController.cs
Services/OrderService.cs
Repositories/OrderRepository.cs
Dtos/OrderDto.cs
Validators/OrderValidator.cs
```

Review:

```markdown
This is still organized by technical layer. For the `CreateOrder` behavior, move the request, response, validator, handler, endpoint mapping, and tests into `Features/Orders/CreateOrder`. Keep `Order` domain behavior in `Ordering/Domain` if it enforces invariants. Avoid reusing `OrderDto` for unrelated endpoints.
```

### Example B: Bad sharing

```csharp
// Features/Payments/CapturePayment/Handler.cs
await _mediator.Send(new CreateShipmentCommand(orderId));
```

Review:

```markdown
This creates a direct dependency from Payments into Shipping's private slice. Prefer publishing `PaymentCaptured` and let Shipping consume it, or create an explicit `OrderFulfillmentProcess` workflow if synchronous orchestration is required.
```

### Example C: Good query slice

```text
Reports/GetMonthlyRevenue/
  Endpoint.cs
  Query.cs
  Handler.cs
  Response.cs
  Sql.cs
  Tests.cs
```

Review:

```markdown
This is a good vertical query slice. It owns its request, response, SQL/projection, and tests. It does not need to load domain aggregates because it is read-only and report-shaped.
```

---

## 44. Summary

Vertical Slice Architecture is a practical way to organize software around behavior. It reduces change scatter by placing feature-specific code together and reduces coupling by discouraging broad shared abstractions. It works especially well with CQRS, but does not require CQRS. It works with Clean Architecture, Hexagonal Architecture, modular monoliths, microservices, and serverless, but does not replace the need for clear boundaries, tests, public contracts, and domain modeling.

The best VSA implementations are not the ones with the prettiest folders. They are the ones where a developer or agent can open a slice, understand the behavior end to end, change it safely, test it quickly, and avoid surprising other parts of the system.
