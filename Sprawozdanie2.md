### Ćwiczenie: Integracja z Azure Bing Search Service


W pierwszej kolejności utworzono oraz skonfigurowano nową grupę zasobów.

![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/e827432c-37ec-4d08-b3c0-575e1aeee84a)


Następnie utworzono zasób Cognitive Services, który obecnie nazywa się 'Azure AI Services'.

![Zrzut ekranu 2023-12-14 121850](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/3aba6aa3-e8d8-4068-9351-e018d0e3f112)


Należało również przygotować środowkisko programistyczne do utworzenia aplikacji. W tym celu pobrano Visual Studio oraz zainstalowano odpowiednie paczki.

![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/e1d613eb-c3c6-4c19-8bad-83dd858e56bb)
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/b32cd09d-4338-4302-9dce-8d3894a0c90b)
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/67df3f96-9c06-4c7a-9168-618d0659a6a5)
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/22d68bd4-c58d-428f-a207-1d57e3bc519b)




Potem przekopiowano klucz dostępu i punkt końcowy oraz wklejono to pliku konfiguracyjnego 'appsettings.json'.

![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/ab78490f-66b8-4cd7-bd66-64732126f933)

```
{
  "AzureCognitiveServices": {
    "BingSearchApiKey": "740a7cf0b45e4dedaf09264ddeff1dfc",
    "BingSearchEndpoint": "https://azurecognitiveservicespuch.cognitiveservices.azure.com/"
  }
}
```

Następnie utworzono plik ConfigurationHelper.cs pełni rolę pomocniczą w zarządzaniu konfiguracją aplikacji.
```
using Microsoft.Extensions.Configuration;
using System.IO;

public class ConfigurationHelper
{
    public static IConfigurationRoot GetConfiguration()
    {
        return new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("appsettings.json")
            .Build();
    }
}
```


Do obsługi wyszukiwań utworzono plik SearchService.cs.

```
using Microsoft.Azure.CognitiveServices.Search.WebSearch;
using Microsoft.Azure.CognitiveServices.Search.WebSearch.Models;
using Microsoft.Extensions.Configuration;
using System;
using System.Threading.Tasks;

public class SearchService
{
    private readonly IConfigurationRoot _configuration;
    private readonly WebSearchClient _webSearchClient;

    public SearchService()
    {
        _configuration = ConfigurationHelper.GetConfiguration();

        var bingSearchApiKey = _configuration["AzureCognitiveServices:BingSearchApiKey"];
        var bingSearchEndpoint = _configuration["AzureCognitiveServices:BingSearchEndpoint"];

        Console.WriteLine($"API Key: {bingSearchApiKey}");
        Console.WriteLine($"Endpoint: {bingSearchEndpoint}");

        _webSearchClient = new WebSearchClient(new ApiKeyServiceClientCredentials(bingSearchApiKey));
        _webSearchClient.Endpoint = bingSearchEndpoint;
    }

    public async Task SearchAsync(string query)
    {
        try
        {
            var webData = await _webSearchClient.Web.SearchAsync(query);

            if (webData != null && webData.WebPages != null && webData.WebPages.Value.Count > 0)
            {
                // Przetwarzanie wyników, np. wyświetlanie ich w konsoli
                foreach (var result in webData.WebPages.Value)
                {
                    Console.WriteLine($"Title: {result.Name}");
                    Console.WriteLine($"URL: {result.Url}\n");
                }
            }
            else
            {
                Console.WriteLine($"Brak wyników dla zapytania: '{query}'.");
            }
        }
        catch (ErrorResponseException ex) when (ex.Response.StatusCode == System.Net.HttpStatusCode.NotFound)
        {
            // Obsługa błędu 404 (NotFound)
            Console.WriteLine($"Nie znaleziono wyników dla zapytania: '{query}'.");
        }
        catch (ErrorResponseException ex)
        {
            // Inne błędy związane z zapytaniem do usługi Bing Search
            Console.WriteLine($"Błąd w zapytaniu do usługi Bing Search: {ex.Message}");
            Console.WriteLine($"Treść odpowiedzi: {ex.Response.Content}");
        }
        catch (Exception ex)
        {
            // Inny ogólny wyjątek
            Console.WriteLine($"Wystąpił inny błąd: {ex.Message}");
        }
        finally
        {
            Console.WriteLine("Naciśnij Enter, aby zakończyć program.");
            Console.ReadLine();
        }
    }



}
```

Kod programu uruchamiającego aplikację wygląda następująco:

```
using System;
using System.Threading.Tasks;

class Program
{
    static async Task Main(string[] args)
    {
        try
        {
            var searchService = new SearchService();

            Console.Write("Wprowadź zapytanie: ");
            var query = Console.ReadLine();

            await searchService.SearchAsync(query);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Wystąpił błąd: {ex.Message}");
        }
        finally
        {
            Console.WriteLine("Naciśnij Enter, aby zakończyć program.");
            Console.ReadLine();
        }
    }
}
```

Pu uruchomieniu programu i wpisaniu zapytania dostajemy niestety błąd 404.
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/bf219edf-8c3a-469d-9743-b08f9541a3fb)



### Ćwiczenie: Praca z Azure Speech Service

Tak jak w poprzednim ćwiczeniu utworzono nowy zasób (tym erazem "Speech Service") oraz dokonano jego konfiguracji. 
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/2edddffa-a6df-4222-9ba4-65ea05b7422b)

Skopiowano poniższy klucz oraz region i przypisano do zmiennych.
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/19c4ebd9-2078-425f-8326-95aca36ecd4a)

Następnie utworzono nowy projekt w C# oraz zaimplementowano kod do obsługi wykrywania głosy.

```
using System;
using System.IO;
using System.Threading.Tasks;
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;

class Program
{
    // This example requires environment variables named "SPEECH_KEY" and "SPEECH_REGION"
    static string speechKey = Environment.GetEnvironmentVariable("SPEECH_KEY");
    static string speechRegion = Environment.GetEnvironmentVariable("SPEECH_REGION");

    static void OutputSpeechRecognitionResult(SpeechRecognitionResult speechRecognitionResult)
    {
        switch (speechRecognitionResult.Reason)
        {
            case ResultReason.RecognizedSpeech:
                Console.WriteLine($"RECOGNIZED: Text={speechRecognitionResult.Text}");
                break;
            case ResultReason.NoMatch:
                Console.WriteLine($"NOMATCH: Speech could not be recognized.");
                break;
            case ResultReason.Canceled:
                var cancellation = CancellationDetails.FromResult(speechRecognitionResult);
                Console.WriteLine($"CANCELED: Reason={cancellation.Reason}");

                if (cancellation.Reason == CancellationReason.Error)
                {
                    Console.WriteLine($"CANCELED: ErrorCode={cancellation.ErrorCode}");
                    Console.WriteLine($"CANCELED: ErrorDetails={cancellation.ErrorDetails}");
                    Console.WriteLine($"CANCELED: Did you set the speech resource key and region values?");
                }
                break;
        }
    }

    async static Task Main(string[] args)
    {
        var speechConfig = SpeechConfig.FromSubscription(speechKey, speechRegion);
        speechConfig.SpeechRecognitionLanguage = "en-US";

        using var audioConfig = AudioConfig.FromDefaultMicrophoneInput();
        using var speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);

        Console.WriteLine("Speak into your microphone.");
        var speechRecognitionResult = await speechRecognizer.RecognizeOnceAsync();
        OutputSpeechRecognitionResult(speechRecognitionResult);
    }
}
```


