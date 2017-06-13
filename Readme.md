# hexagonalThis
A simple kata to live-code with [__Alistair__](http://alistair.cockburn.us/) about __Hexagonal Architecture__


![HexagonalTitle](https://github.com/tpierrain/hexagonalThis/blob/confCallWithAlistair/HexagonalInANutshell.png?raw=true)


## Ports? Adapters?

### Ports
A port matches one intention (usually an interface in C# or java) but may have multiple verbs/methods. e.g.:

````CSharp

    // The 1st port to ask/enter the hexagon
    public interface IProvideVerses
    {
        string GiveMeSomePoetry();
        string GiveMeLinesOfPoetry(int numberOfLine);
    }


    // The 1st port to exit the hexagon 
    // (by looking for some poetry outside of it)
    public interface IKnowABunchOfPoetry
    {
        string GetPoem();
    }

````


![warning](https://github.com/tpierrain/hexagonalThis/blob/confCallWithAlistair/warning.png?raw=true)  __Note: All ports belong to the Domain side__


### Adapters
__An Adapter is something which allows to enter/exit the hexagon__. An Adapter is both our code which maps the infra <-> domain data models (i.e. in the GoF adapter pattern sense) and everything that allows us to enter/exit the hexagon (e.g.: a database driver, your favorite HTTP stack, etc).

![warning](https://github.com/tpierrain/hexagonalThis/blob/confCallWithAlistair/warning.png?raw=true)  __Note: All adapters belong to the Infra(structure) side__



Here, we have 1 Adapter so far:
 - [__ConsoleAdapter__](https://github.com/tpierrain/hexagonalThis/blob/confCallWithAlistair/HexagonalThis.Console/Adapters/ConsoleAdapter.cs)

Here is a typical method from this Adapter:

````CSharp

    public void RequestFirstVersesForAPoem(string line)
    {
        // Map the request passed by the infra side (here Console app arguments)
        int numberOfLine = int.Parse(line);

        // Call the business logic
        var verses = this.poet.GiveMeLinesOfPoetry(numberOfLine);

        // Get back with the answer onto the Infra side (Console)
        this.publicationStrategy.WriteLine(verses);
    }

````

## The Hexagon

The hexagon must expose one or multiple ports. Here, the [__Poet__](https://github.com/tpierrain/hexagonalThis/blob/confCallWithAlistair/HexagonalThis.Domain/Poet.cs#L13) hexagon implements the __IProvideVerses__ port and use the __IKnowABunchOfPoetry__ port to ask an external service some poetry to work with.

![Hexagon](https://github.com/tpierrain/hexagonalThis/blob/confCallWithAlistair/HexagonalThis.png?raw=true)


## A 3 steps initialization

![Hexagon](https://github.com/tpierrain/hexagonalThis/blob/confCallWithAlistair/A3StepsInitialization.PNG?raw=true)

Below, a typical hexagonal architecture initialization:


````CSharp

    // ASP.NET Startup class...

    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        // Add framework services.
        services.AddMvc();

        // 1. Instantiate adapters to go out
        var trainDataService = new TrainDataService(UriTrainDataService);
        var bookingReferenceService = new BookingReferenceService(UriBookingReferenceService);

        // 2. Instantiate the hexagon (here, the usage of hexagonal 
        //    architecture is made explicit with a thin 'Hexagon' wrapper)
        var hexagon = new Hexagon(trainDataService, bookingReferenceService);

        // 3. Instantiate the "I need to enter/ask" adapter
        var reserveSeatsAdapter = new ReserveSeatsRestAdapter(hexagon);

        // All your application keeps is a reference to the "I need to enter/ask" adapters (here registered as singleton within the ASP.NET container)
        services.AddSingleton<ReserveSeatsRestAdapter>(reserveSeatsAdapter);
    }

````