# AlgorizaApp - Currency Exchange Service

## Project Overview

AlgorizaApp is a comprehensive .NET Core application designed to manage currency exchanges with a robust multi-layered architecture. This service allows users to manage currencies, perform currency exchanges, and maintain a history of exchange transactions.

## Architecture

The solution is structured using a clean architecture approach with 5 distinct layers:

### Layer Structure

- **CoreLayer**: Contains domain entities, interfaces, and business logic rules
- **RepositryLayer**: Handles data persistence operations and database interactions
- **ServicesLayer**: Implements business logic and service interfaces
- **UnitOfWorkLayer**: Manages transaction coordination across multiple repositories
- **AlgorizaAPI**: Exposes RESTful API endpoints for client applications

## Key Features

### Currency Management

- Create and manage multiple currency definitions
- Enable/disable currencies with active status tracking
- Fast currency lookup by name
- Complete CRUD operations for currency maintenance

### Exchange Operations

- Perform currency exchanges with current rates
- Validate exchange parameters before processing
- Calculate exchange amounts automatically

### Transaction History

- Record all exchange operations for auditing purposes
- Track historical exchange rates and volumes
- Query exchange history by various parameters

### Identity and Authentication

- Secure user identity management
- Authentication and authorization for API access

## Technical Implementation

### Repository Pattern

The application implements the Repository pattern to abstract data access logic:

```csharp
// CurrencyService implements repository pattern for Currency entities
public class CurrencyService : Repositry<Currency>, ICurrencyService
{
    public CurrencyService(AppDbContext appDbContext)
        : base(appDbContext)
    {
    }

    // Repository operations for Currency entities
    public void AddCurrency(Currency currency)
    {
        var oldcurrency = GetCurrencyByName(currency.Name);
        if (oldcurrency != null)
        {
            oldcurrency.IsActive = true;
        }
        else
        {
            currency.Id = 0;
            AppDbContext.Currencies.Add(currency);
        }
    }

    public IEnumerable<Currency> GetAllCurrencies()
    {
        return AppDbContext.Currencies.Where(e => e.IsActive == true);
    }

    public Currency GetCurrencyByName(string Name)
    {
        return AppDbContext.Currencies.FirstOrDefault(e => e.Name == Name);
    }
}
```

### Unit of Work Pattern

The UnitOfWork pattern coordinates operations across multiple repositories and ensures data consistency:

```csharp
public class UnitOfWork : IUnitOfWork
{
    private readonly AppDbContext AppDbContext;

    public UnitOfWork(AppDbContext appDbContext)
    {
        AppDbContext = appDbContext;
        Currencies = new CurrencyService(AppDbContext);
        ExchangeHistories = new ExchangeHistoryService(AppDbContext);
        Identites = new IdentityService(AppDbContext);
    }

    public ICurrencyService Currencies { get; set; }
    public IExchangeHistoryService ExchangeHistories { get; set; }
    public IIdentityService Identites { get; set; }

    public int Complete()
    {
        return AppDbContext.SaveChanges();
    }

    public void Dispose()
    {
        AppDbContext.Dispose();
    }
}
```

## Database Design

The application uses Entity Framework Core with a SQL Server database. Key entities include:

### Currency

- Id: Unique identifier
- Name: Currency name (e.g., USD, EUR)
- Symbol: Currency symbol
- IsActive: Status flag for active currencies

### ExchangeHistory

- Id: Unique identifier
- FromCurrencyId: Source currency
- ToCurrencyId: Target currency
- Rate: Exchange rate at transaction time
- Amount: Amount exchanged
- ResultAmount: Calculated result amount
- TransactionDate: Date and time of exchange

## Getting Started

### Prerequisites

- .NET Core SDK 6.0 or later
- SQL Server 2019 or later
- Visual Studio 2022 (recommended)

### Installation Steps

1. Clone the repository

```bash
git clone https://github.com/yourusername/AlgorizaApp.git
```

2. Navigate to the solution directory and restore packages

```bash
cd AlgorizaApp
dotnet restore
```

3. Update the database connection string in `appsettings.json` within the AlgorizaAPI project

4. Apply migrations to create the database

```bash
dotnet ef database update --project AlgorizaAPI
```

5. Build and run the application

```bash
dotnet build
dotnet run --project AlgorizaAPI
```

6. Access the API at `https://localhost:5001` or the configured port

## API Endpoints

### Currency Management

- `GET /api/currencies` - Retrieve all active currencies
- `GET /api/currencies/{name}` - Get currency by name
- `POST /api/currencies` - Create a new currency
- `PUT /api/currencies/{id}` - Update currency details
- `DELETE /api/currencies/{id}` - Deactivate a currency

### Exchange Operations

- `POST /api/exchange` - Perform a currency exchange
- `GET /api/exchange/history` - Retrieve exchange history
- `GET /api/exchange/rates` - Get current exchange rates

## Development Guidelines

### Adding New Features

1. Define domain entities in CoreLayer
2. Create repository interfaces in RepositryLayer
3. Implement services in ServicesLayer
4. Update UnitOfWork to include new services
5. Add API controllers in AlgorizaAPI project

### Running Tests

```bash
dotnet test
```

### Code Style

This project follows Microsoft's .NET coding conventions. Use an editor with EditorConfig support to maintain consistent code style.

## Deployment

### Production Environment Setup

1. Configure environment-specific settings in `appsettings.Production.json`
2. Deploy using Azure DevOps pipeline or similar CI/CD tool
3. Set up SQL Server database with appropriate security measures
4. Configure application monitoring and logging

## License

[MIT](https://choosealicense.com/licenses/mit/)

## Contributors

- [Mostafa Elsaeed](mailto:mostafaelsaeed1544@gmail.com)

## Acknowledgments

- Thanks to Algoriza for the project requirements and guidance
