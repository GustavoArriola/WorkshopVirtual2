# Introducción a Change Feed

En este apartado vamos a ver como trabajar con el Chang Feed de Cosmos DB.  
Con Change Feed puedes reaccionar a los cambios en tu Cosmos DB y realizar operaciones paralelas  

## Creando change feed en proyecto consola 

1- En la carpeta donde quieras construir el lab abrir Visual Studio Code, para ello abrir la consola en el directorio desaado y ejecutar:   
> code .  

2- Abrir el terminal de vscode: 
> Ctrl+Shift+ñ o Terminal - New Terminal  

3 - Creamos el proyecto de consola:  
> dotnet new console -o "netcoreconfvirtualworkshopchangefeed"  
> cd netcoreconfvirtualworkshopchangefeed

4- Añadimos el sdk de Cosmos DB a nuestro proyecto  
> dotnet add package Microsoft.Azure.Cosmos -v 3.13.0 

## Modelo de datos de Cosmos  

Creamos una clase llamada AvengersModel.cs que contendrá nuestro modelo de Cosmos DB, y lo rellenamos con el siguiente modelo:  

```csharp

    using System;  
    using System.Collections.Generic; 
    
    namespace netcoreconfvirtualworkshopchangefeed
    {
        public class AvengersModel
        {
            public string id { get; set; } 
            public string title { get; set; } 
            public string description { get; set; } 
            public string resourceURI { get; set; } 
            public List<Url> urls { get; set; } 
            public int startYear { get; set; } 
            public int endYear { get; set; } 
            public string rating { get; set; } 
            public string type { get; set; } 
            public DateTime modified { get; set; } 
            public Thumbnail thumbnail { get; set; } 
            public Creators creators { get; set; } 
            public Characters characters { get; set; } 
            public Stories stories { get; set; } 
            public Comics comics { get; set; } 
            public Events events { get; set; } 
            public object next { get; set; } 
            public object previous { get; set; } 

        }

        public class Url    {
            public string type { get; set; } 
            public string url { get; set; } 
        }

        public class Thumbnail    {
            public string path { get; set; } 
            public string extension { get; set; } 
        }

        public class Item    {
            public string resourceURI { get; set; } 
            public string name { get; set; } 
            public string role { get; set; } 
        }

        public class Creators    {
            public int available { get; set; } 
            public string collectionURI { get; set; } 
            public List<Item> items { get; set; } 
            public int returned { get; set; } 
        }

        public class Item2    {
            public string resourceURI { get; set; } 
            public string name { get; set; } 
        }

        public class Characters    {
            public int available { get; set; } 
            public string collectionURI { get; set; } 
            public List<Item2> items { get; set; } 
            public int returned { get; set; } 
        }

        public class Item3    {
            public string resourceURI { get; set; } 
            public string name { get; set; } 
            public string type { get; set; } 
        }

        public class Stories    {
            public int available { get; set; } 
            public string collectionURI { get; set; } 
            public List<Item3> items { get; set; } 
            public int returned { get; set; } 
        }

        public class Item4    {
            public string resourceURI { get; set; } 
            public string name { get; set; } 
        }

        public class Comics    {
            public int available { get; set; } 
            public string collectionURI { get; set; } 
            public List<Item4> items { get; set; } 
            public int returned { get; set; } 
        }

        public class Events    {
            public int available { get; set; } 
            public string collectionURI { get; set; } 
            public List<object> items { get; set; } 
            public int returned { get; set; } 
        }
    }
```  

## Añadiendo código Change Feed 

Cambiamos el código de Progam.cs por este:  

```csharp  
using System;
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Azure.Cosmos;

namespace netcoreconfvirtualworkshopchangefeed
{
    class Program
    {
       private static readonly string _endpointUrl = "";
        private static readonly string _primaryKey = "";
        private static readonly string _databaseId = "marvel";
        private static readonly string _containerId = "series";
        private static readonly string _destinationContainerId = "seriesCopy";
        private static CosmosClient _client = new CosmosClient(_endpointUrl, _primaryKey);

        static async Task Main(string[] args)
        {

            Database db = _client.GetDatabase(_databaseId);
            Container container = db.GetContainer(_containerId);
           
            ContainerProperties destinationContainerProperties = new ContainerProperties(_destinationContainerId, "/type");
            Container destinationContainer = await  db.CreateContainerIfNotExistsAsync(destinationContainerProperties, throughput: 400);

             ContainerProperties leaseContainerProperties = new ContainerProperties("leases", "/id");
            Container leaseContainer = await  db.CreateContainerIfNotExistsAsync(leaseContainerProperties, throughput: 400);


            var builder = container.GetChangeFeedProcessorBuilder("consolefeed",
               (IReadOnlyCollection<AvengersModel> input, CancellationToken cancellationToken) =>
               {
                  Console.WriteLine(input.Count + " Changes Received");
                  var tasks = new List<Task>();

                  foreach (var model in input)
                  {
                    tasks.Add(destinationContainer.CreateItemAsync(model, new PartitionKey(model.type)));
                  }

                  return Task.WhenAll(tasks);
               });

            var processor = builder
                            .WithInstanceName("changeFeedConsole")
                            .WithLeaseContainer(leaseContainer)
                            .Build();

            await processor.StartAsync();
            Console.WriteLine("Started Change Feed Processor");
            Console.WriteLine("Press any key to stop the processor...");

            Console.ReadKey();

            Console.WriteLine("Stopping Change Feed Processor");
            await processor.StopAsync();
        }
    }
}
``` 
En _databaseId y _containerId poner el nombre que hayais asigando en la creación del Cosmos DB y ejecutamos:  
> dotnet build  
> dotnet run  

Ahora para ver que funciona vamos al portal y cargamos el fichero avengers.json  
Para verificar que todo ha ido bien, ir al port y ver que se han creado las colecciones seriesCopy y leases.  

## Change feed en Azure functions  

1- En la carpeta donde quieras construir el lab abrir Visual Studio Code, para ello abrir la consola en el directorio desaado y ejecutar:   
> code .  

2- Abrir el terminal de vscode: 
> Ctrl+Shift+ñ o Terminal - New Terminal  

3- Vamos a necesitar node.js para ejecutar azure functions

4- En el terminal verificar la versión de node:  
>node --version  
>Si no tienes instalado node.js lo puedes instalar [aquí](https://docs.npmjs.com/getting-started/installing-node#osx-or-windows)

5- Si lo tenemos instalado instalaremos la última versión  
>npm i -g node@latest

6- Bajamos las tools de Azure Functions  
>npm install -g azure-functions-core-tools 

7- Ahora creamo la Azure Function
>func init ChangeFeedFunction
>cd ChangeFeedFunction

8- Ahora ejecutamos el siguiente comando_
>func new
>Seleccionamos **C#**  
>Seleccionamos **CosmosDBTrigger**  
>Como nombre de la function ponemos netcoreconf  

9- Abrimos el fichero ChangeFeedFunction.cs y cambiamos el framework a 3.1
>TargetFramework>netcoreapp3.1</TargetFramework>

10- Ahora añadimos los siguientes paquetes nuget 
>dotnet add package Microsoft.Azure.Cosmos  
>dotnet add package Microsoft.NET.Sdk.Functions --version 3.0.9  
>dotnet add package Microsoft.Azure.WebJobs.Extensions.CosmosDB --version 3.0.7  

11- En el fichero local.settings.json añadimos la entrada CosmosConnections:

```json  
{
    "IsEncrypted": false,
    "Values": {
        "AzureWebJobsStorage": "UseDevelopmentStorage=true",
        "FUNCTIONS_WORKER_RUNTIME": "dotnet",
        "CosmosConnections": ""
    }
}
``` 
>En CosmosConnections añadir la connection string del Cosmos DB

12- Añadimos la clase AvengersModel.cs: 
```csharp

    using System;  
    using System.Collections.Generic; 
    
    namespace ChangeFeedFunction
    {
        public class AvengersModel
        {
            public string id { get; set; } 
            public string title { get; set; } 
            public string description { get; set; } 
            public string resourceURI { get; set; } 
            public List<Url> urls { get; set; } 
            public int startYear { get; set; } 
            public int endYear { get; set; } 
            public string rating { get; set; } 
            public string type { get; set; } 
            public DateTime modified { get; set; } 
            public Thumbnail thumbnail { get; set; } 
            public Creators creators { get; set; } 
            public Characters characters { get; set; } 
            public Stories stories { get; set; } 
            public Comics comics { get; set; } 
            public Events events { get; set; } 
            public object next { get; set; } 
            public object previous { get; set; } 

        }

        public class Url    {
            public string type { get; set; } 
            public string url { get; set; } 
        }

        public class Thumbnail    {
            public string path { get; set; } 
            public string extension { get; set; } 
        }

        public class Item    {
            public string resourceURI { get; set; } 
            public string name { get; set; } 
            public string role { get; set; } 
        }

        public class Creators    {
            public int available { get; set; } 
            public string collectionURI { get; set; } 
            public List<Item> items { get; set; } 
            public int returned { get; set; } 
        }

        public class Item2    {
            public string resourceURI { get; set; } 
            public string name { get; set; } 
        }

        public class Characters    {
            public int available { get; set; } 
            public string collectionURI { get; set; } 
            public List<Item2> items { get; set; } 
            public int returned { get; set; } 
        }

        public class Item3    {
            public string resourceURI { get; set; } 
            public string name { get; set; } 
            public string type { get; set; } 
        }

        public class Stories    {
            public int available { get; set; } 
            public string collectionURI { get; set; } 
            public List<Item3> items { get; set; } 
            public int returned { get; set; } 
        }

        public class Item4    {
            public string resourceURI { get; set; } 
            public string name { get; set; } 
        }

        public class Comics    {
            public int available { get; set; } 
            public string collectionURI { get; set; } 
            public List<Item4> items { get; set; } 
            public int returned { get; set; } 
        }

        public class Events    {
            public int available { get; set; } 
            public string collectionURI { get; set; } 
            public List<object> items { get; set; } 
            public int returned { get; set; } 
        }
    }
```  

13 - Cambiamos el fichero netcoreconf.cs
```csharp
using System;
using System.Collections.Generic;
using Microsoft.Azure.Documents;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;

namespace ChangeFeedFunction
{
    public static class netcoreconf
    {
        [FunctionName("netcoreconf")]
        public static void Run([CosmosDBTrigger(
            databaseName: "marvel",
            collectionName: "series",
            ConnectionStringSetting = "CosmosConnections",
            CreateLeaseCollectionIfNotExists = true,
            LeaseCollectionName = "leases")]IReadOnlyList<Document> input, ILogger log)
        {
            foreach(var item in input)
            {
                var model = JsonConvert.DeserializeObject<AvengersModel>(item.ToString());
                Console.WriteLine($"{model.title}");
            }
        }
    }
}

``` 
14- Ahora ejecutamos la funcion:  
>dotnet build  
>func host start  
>Cargams en el cosmos el fichero avengers2.json  




