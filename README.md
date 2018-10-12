# <img align="center" src="./docs/img/polyright-icon.png" height="64">  polyright SDK

The Polyright SDK simplifies the integration of polyright functionalities into a third-party system. All necessary methods to authenticate user, read cards and do polyright or TWINT transactions through the polyright platform are provided.
The security is managed on the Polyright platform.


## Functionalities

### Device Authentication

The SDK allows activation token management. That is: 
- Get a token to activate any device through the Polyright platform
- Send device information to the Polyright platform
- Automatically retry at regular interval until activated
- Store and automatically renew the authentication token


### Card Reader management

-	Initialization of all plugged card readers
  - Automatically get the readers configuration from affiliation polyright system
  - Multiple reader types (Legic2000, Legi4000, NFC)
-	Read the card currently tapped on the reader
-	Raised events (Any reader ready / no reader ready)


### TWINT Beacon management

-	Seamless TWINT beacons initialization
-	Automatic firmware update
-	Ready to pay with TWINT
  - TWINT payment itself is managed by the polyright platform


### Customer information
-	Wait for a card and get owner information (all person properties and custom informations)
-	Get the transactions of the person

### Financial
- Transaction process
  - Begin a transaction with information (amount, description, sold products, ...)
  - Wait for a customer (polyright card , TWINT customer)
  - Choose the person account to use
  - Wait until the transaction is finalized
  - Cancel the transaction
  
- Financial closing
  - Do a financial closing and get a summary of the transactions done

- Get the transactions done by the device

# Getting Started

## Prerequisites
- RFID or NFC reader (for polyright payments)
- TWINT beacon (for TWINT payment with beacon)
- Internet connection
- Access to the Polyright nuget repository : https://nuget.polyright.com
- Compatible platforms:
  - .NET 4.5
  - Universal Windows Platform (UWP)
  - Android (with Xamarin)
  - IOS 12 (with Xamarin)

## Installation
1. Install the Polyright SDK NuGet package

## Initialization

```
var cardReaders = new Polyright.SDK.Devices.CardReaders(new CardReaders());
var beacons = new Polyright.SDK.Devices.Beacons(new Beacons());
var clientMetadata = new ClientMetadata
{
	ApplicationName = "MyTestApplication",
	ApplicationVersion = "1.0.0.0"
};
Polyright.SDK.Windows.Container.Init(cardReaders, beacons, null, clientMetadata);
PolyrightContext.Init(options =>
{
  options.ClientId = "Polyright.PaymentSDK";
  options.SecretKey = "f952a0a18be81faaafd842e24232f96c8052ede6328601d06c9f10a0130b7f7f";
  options.EnvironmentType = EnvironmentType.Development;
});

```
## Authentication

```
var connectionManager = new ConnectionManager();
connectionManager.ConnectAsync(eventArgs =>
{
   Console.WriteLine($"Activation code: {eventArgs.ActivationCode}");
   return Task.CompletedTask;
});
await connectionManager.AwaitServiceReadyAsync(token);
```

## Wait for devices (card reader, TWINT beacon)

```
var deviceManager = new DeviceManager();
await deviceManager.AwaitCardReaderReadyAsync(token);
await deviceManager.AwaitTwintBeaconReadyAsync(token);

```

## Do a transaction

```
var financialService = new FinancialService();
var transactionRequest = new TransactionRequest
{
	Amount = -1m,
	Purpose = "Transaction purpose"
}.OnSelectAccount((scope, persons) =>
{
// Select person account. In this example, select use the personal account
  var person = persons.FirstOrDefault();
  IAccount account = person?.PersonalAccount;
  //Console.WriteLine($"{person?.FormattedName}: {account?.Balance}");
  return Task.FromResult(account);
});
var transactionScope = await financialService.BeginTransactionAsync(transactionRequest, token);
await transactionScope.AwaitTransactionCompletionAsync(token);
Console.WriteLine($"Transaction completed. Status: {transactionScope.Transaction.Status}");
```
