# C#, .NET & Java Prompts — Production-Grade AI Prompt Templates

> Curated prompts for C#/.NET and Java enterprise development, web APIs, desktop, and mobile. Based on the [CRTSE framework](best-practices.md).

---

## C# / .NET

### ASP.NET Core Minimal API
```
Act as a principal .NET engineer (15 years C#, enterprise API specialist).

I'm building a .NET 9 minimal API for a warehouse inventory system.

Task: Create the `GET /inventory/{sku}` endpoint that returns stock levels with warehouse location details.

Requirements:
- Use .NET 9 minimal API (not traditional controllers)
- Return `Results<Ok<InventoryResponse>, NotFound>` typed result
- Use a record for the response DTO (not class)
- Query Entity Framework Core 9 with compiled queries for performance
- Include warehouse name and aisle/bin location in the response
- Add `ETag` response header for caching (hash of last-modified timestamp)
- Add OpenAPI metadata ([Tags], [ProducesResponseType] attributes)
- Use `ILogger<T>` with structured logging (message template, not string interpolation)

Do NOT:
- Return `IActionResult` — use typed results
- Use `async void` anywhere
- Eager-load the entire warehouse table (use `.Select()` projection)
- Expose internal IDs or database timestamps in the response

Our existing endpoint pattern:
```csharp
app.MapGet("/api/items/{id}", async (string id, InventoryDb db, ILogger<Program> logger) =>
    await db.Items.Where(i => i.Sku == id).Select(i => new ItemResponse(
        i.Sku, i.Name, i.QuantityOnHand, i.Warehouse.Name
    )).FirstOrDefaultAsync()
    is ItemResponse item ? Results.Ok(item) : Results.NotFound());
```

Output: the complete endpoint with its DTO records, EF query, and any middleware configuration needed.
```

### Entity Framework Core migration + performance
```
Act as a senior .NET backend engineer specializing in database performance.

Our `Orders` table has grown to 2M rows and queries filtering by `CustomerId + OrderDate` are slow.

Task: Analyze and fix the performance issue — suggest the migration, the EF model change, and any query rewrite needed.

Requirements:
- Identify the missing index(es) with the exact columns and order
- Write the EF migration to add the index (ensure the migration is idempotent — check if index exists first)
- Show the Before and After query with EF compiled query syntax
- Add pagination (cursor-based, not offset-based) using `OrderDate` as the cursor

Do NOT:
- Add indexes without analyzing the query plan first
- Use `Skip()/Take()` for pagination — that's offset-based and degrades on deep pages

Output: the migration file, updated DbContext, and optimized query method.
```

### WPF MVVM with ObservableObject
```
Act as a senior .NET desktop application engineer.

I'm building a WPF application (.NET 9, CommunityToolkit.Mvvm) for a customer service dashboard.

Task: Implement the `CustomerSearchViewModel` that:
- Binds to a search text box and a results DataGrid
- Debounces search input (300ms after last keystroke)
- Calls `ICustomerService.SearchAsync(string query, CancellationToken ct)` 
- Cancels in-flight searches when a new keystroke arrives
- Displays results in an `ObservableCollection<CustomerResult>`
- Shows a `bool IsSearching` property for loading indicator binding
- Shows `bool HasNoResults` when search completes with empty results
- Handles errors with an `ObservableProperty<string?> ErrorMessage` that auto-clears after 5 seconds

Requirements:
- Use CommunityToolkit.Mvvm source generators (`[ObservableProperty]`, `[RelayCommand]`)
- Search cancellation must NOT throw `OperationCanceledException` — just suppress
- The search command must accept a string parameter (the search text)

Do NOT:
- Use code-behind for any behavior — all logic in the ViewModel
- Use `async void` — all commands are `async Task`
- Bind directly to `List<T>` — use `ObservableCollection`

Output: the complete ViewModel class.
```

### Unity game script (C#)
```
Act as a senior Unity developer (C#, Unity 6, game systems specialist).

I'm building a 3D collectible card game with drag-and-drop card mechanics.

Task: Implement the `CardDragHandler` MonoBehaviour that:
- Detects drag start on a Card GameObject (via IPointerDownHandler)
- Moves the card with the pointer during drag (world space, following the pointer raycast to a "table" layer)
- Snaps the card to a grid slot on drop (plays snap animation via DOTween)
- If dropped outside a valid zone, the card returns to its original position (spring-back animation)
- Highlights valid drop zones (glow shader property) while dragging
- Works on both mouse and touch input (Unity's Input System package)

Requirements:
- Use the Unity Input System (not legacy Input Manager)
- Use DOTween for all animations (no custom coroutine tweens)
- Respect the card's z-sorting layer during drag (render above other cards)
- On drop, fire a `UnityEvent<CardDropData>` so other systems can react

Do NOT:
- Use `Update()` for continuous polling — use event-driven input
- Hardcode grid positions — read from a `GridLayout` component
- Use `GameObject.Find()` or string-based lookups

Output: the complete `CardDragHandler.cs` script.
```

---

## Java

### Spring Boot REST API
```
Act as a senior Java backend engineer (10+ years, Spring Boot specialist).

I'm building a Spring Boot 3.4 microservice (Java 21, virtual threads enabled) for a payment processing system.

Task: Implement the `POST /api/payments` endpoint that initiates a payment.

Requirements:
- Use Spring Boot 3.4 with virtual threads
- Accept `@Valid` request body with Jakarta Bean Validation annotations
- Use a record for the request DTO: paymentId, amount (BigDecimal, positive), currency (ISO 4217), paymentMethod (enum: CARD, BANK_TRANSFER, WALLET)
- The response is 202 Accepted with a tracking URL in the Location header
- Payment processing is asynchronous — queue to a message broker, don't block the HTTP thread
- Use `@ControllerAdvice` for centralized exception handling (validation errors → 400, payment not found → 404, gateway timeout → 504)
- Include OpenAPI/Swagger annotations
- Log with SLF4J using structured key=value pairs (not string interpolation)

Do NOT:
- Use `@Autowired` field injection — use constructor injection
- Throw raw `RuntimeException` — use domain-specific custom exceptions
- Use `.get()` on futures inside the controller thread
- Expose stack traces in error responses to clients

Our existing endpoint pattern:
```java
@RestController
@RequestMapping("/api")
public class PaymentController {
    private final PaymentService paymentService;
    private final PaymentDtoMapper mapper;

    // constructor injection

    @PostMapping("/payments")
    public ResponseEntity<Void> initiatePayment(@Valid @RequestBody PaymentRequest request) {
        var trackingId = paymentService.initiate(mapper.toCommand(request));
        return ResponseEntity.accepted()
            .location(URI.create("/api/payments/" + trackingId))
            .build();
    }
}
```

Output: the controller, request/response DTOs, exception handler, and service interface.
```

### Stream processing with Kafka
```
Act as a principal Java engineer specializing in event-driven architectures.

We process financial transactions through Kafka. A new requirement: detect potentially fraudulent transactions in real-time.

Task: Implement a Kafka Streams topology that:
- Consumes from `transactions` topic (key: accountId, value: TransactionEvent)
- For each transaction, compute a 5-minute sliding window of:
  - Total amount sum
  - Distinct merchant count
  - Transaction count
- If any window metric exceeds its threshold, emit an `AlertEvent` to the `fraud-alerts` topic
- Thresholds are configurable per account (read from a compacted `account-config` topic)

Requirements:
- Use Kafka Streams DSL (not Processor API unless necessary)
- State stores must be persistent (RocksDB) with changelog topics
- Use exactly-once semantics (`processing.guarantee: exactly_once_v2`)
- Graceful shutdown: flush state stores before closing
- Expose window state via a REST endpoint for debugging (`GET /debug/window/{accountId}`)

Do NOT:
- Use global state stores (per-account thresholds should be local to the partition)
- Skip processing if an account config isn't loaded yet (buffer and retry)
- Use `Thread.sleep()` for timing — use punctuators

Output: the Kafka Streams topology class and its configuration.
```

### Android — Jetpack Compose screen
```
Act as a senior Android engineer (Kotlin, Jetpack Compose, modern Android architecture).

I'm building a recipe discovery app.

Task: Implement the `RecipeSearchScreen` composable:
- A search bar at the top (TextField with search icon)
- Debounced search (400ms) calling the ViewModel's search method
- Results displayed in a LazyVerticalGrid (2 columns)
- Each card shows: image (AsyncImage from Coil), title, cook time, difficulty badge
- Pull-to-refresh gesture
- Loading state: shimmer placeholder animation
- Error state: retry button with error message
- Empty state: illustration + "Try searching for something delicious"

Requirements:
- Use Compose with Material 3
- ViewModel with `StateFlow<SearchUiState>` (sealed interface: Loading, Success, Error, Empty)
- Image loading with Coil 3
- Support edge-to-edge display (system bars insets)
- Accessibility: content descriptions on all images and interactive elements

Do NOT:
- Use `LiveData` or `MutableState` outside the ViewModel
- Hardcode string resources (use stringResource)
- Block the main thread with network calls (all coroutine-based in ViewModel)

Output: the Screen composable, ViewModel, and UiState sealed interface.
```