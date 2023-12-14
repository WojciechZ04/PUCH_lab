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
Po uruchomieniu programu wyświetla się informacja żeby powiedzieć coś do mikrofonu.
Po wypowiedzeniu zdania aplikacja się zatrzymuje oraz wyświetla wykryty tekst.
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/27495f83-a566-47ed-9f2e-018ab0238108)

Do konwersji tekstu na mowę utworzono nową aplikację, jednak korzysta ona z tych samych zasobów orach kluczy co aplikacja do konwersji mowy na tekst.
Poniżej kod aplikacji.

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

    static void OutputSpeechSynthesisResult(SpeechSynthesisResult speechSynthesisResult, string text)
    {
        switch (speechSynthesisResult.Reason)
        {
            case ResultReason.SynthesizingAudioCompleted:
                Console.WriteLine($"Speech synthesized for text: [{text}]");
                break;
            case ResultReason.Canceled:
                var cancellation = SpeechSynthesisCancellationDetails.FromResult(speechSynthesisResult);
                Console.WriteLine($"CANCELED: Reason={cancellation.Reason}");

                if (cancellation.Reason == CancellationReason.Error)
                {
                    Console.WriteLine($"CANCELED: ErrorCode={cancellation.ErrorCode}");
                    Console.WriteLine($"CANCELED: ErrorDetails=[{cancellation.ErrorDetails}]");
                    Console.WriteLine($"CANCELED: Did you set the speech resource key and region values?");
                }
                break;
            default:
                break;
        }
    }

    async static Task Main(string[] args)
    {
        var speechConfig = SpeechConfig.FromSubscription(speechKey, speechRegion);

        // The language of the voice that speaks.
        speechConfig.SpeechSynthesisVoiceName = "en-US-JennyNeural";

        using (var speechSynthesizer = new SpeechSynthesizer(speechConfig))
        {
            // Get text from the console and synthesize to the default speaker.
            Console.WriteLine("Enter some text that you want to speak >");
            string text = Console.ReadLine();

            var speechSynthesisResult = await speechSynthesizer.SpeakTextAsync(text);
            OutputSpeechSynthesisResult(speechSynthesisResult, text);
        }

        Console.WriteLine("Press any key to exit...");
        Console.ReadKey();
    }
}
```

Po uruchomieniu, aplikacja prosi o podanie tekstu, który ma zostać wypowiedziany.

![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/1a38a469-cc40-4faa-98f9-9a73612df6f3)

W przypadku zmiany głosu który odczytuje nasz tekst wystarczy zamienić zmienną 
```
speechConfig.SpeechSynthesisVoiceName = "es-ES-ElviraNeural";
```


### Ćwiczenie: Wprowadzenie do Azure Form Recognizer

Ponownie należy dodać nowy zasób oraz go skonfigurować.
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/9c218cad-0fd5-47aa-a085-1795f46b4196)

I pobrać klucze.
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/5e3f6a2e-6073-4d0c-86b4-10edc1294e79)

Następnie nowy projekt w Visual Studio oraz pobrano odpowiednią paczkę z NuGet.
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/dbfc86e9-23d4-43b2-9d79-d2e94aa04bd4)

Oraz utworzono aplikację następującym kodem:
```
using Azure;
using Azure.AI.FormRecognizer.DocumentAnalysis;


string endpoint = "https://formrecognizerpuch.cognitiveservices.azure.com/";
string key = "18d13fd7006c4453b6ba2e4516c5512f";
AzureKeyCredential credential = new AzureKeyCredential(key);
DocumentAnalysisClient client = new DocumentAnalysisClient(new Uri(endpoint), credential);


//sample document
Uri fileUri = new Uri("https://www.w3.org/WAI/WCAG21/working-examples/pdf-table/table.pdf");

AnalyzeDocumentOperation operation = await client.AnalyzeDocumentFromUriAsync(WaitUntil.Completed, "prebuilt-document", fileUri);

AnalyzeResult result = operation.Value;

Console.WriteLine("Detected key-value pairs:");

foreach (DocumentKeyValuePair kvp in result.KeyValuePairs)
{
    if (kvp.Value == null)
    {
        Console.WriteLine($"  Found key with no value: '{kvp.Key.Content}'");
    }
    else
    {
        Console.WriteLine($"  Found key-value pair: '{kvp.Key.Content}' and '{kvp.Value.Content}'");
    }
}

foreach (DocumentPage page in result.Pages)
{
    Console.WriteLine($"Document Page {page.PageNumber} has {page.Lines.Count} line(s), {page.Words.Count} word(s),");
    Console.WriteLine($"and {page.SelectionMarks.Count} selection mark(s).");

    for (int i = 0; i < page.Lines.Count; i++)
    {
        DocumentLine line = page.Lines[i];
        Console.WriteLine($"  Line {i} has content: '{line.Content}'.");

        Console.WriteLine($"    Its bounding box is:");
        Console.WriteLine($"      Upper left => X: {line.BoundingPolygon[0].X}, Y= {line.BoundingPolygon[0].Y}");
        Console.WriteLine($"      Upper right => X: {line.BoundingPolygon[1].X}, Y= {line.BoundingPolygon[1].Y}");
        Console.WriteLine($"      Lower right => X: {line.BoundingPolygon[2].X}, Y= {line.BoundingPolygon[2].Y}");
        Console.WriteLine($"      Lower left => X: {line.BoundingPolygon[3].X}, Y= {line.BoundingPolygon[3].Y}");
    }

    for (int i = 0; i < page.SelectionMarks.Count; i++)
    {
        DocumentSelectionMark selectionMark = page.SelectionMarks[i];

        Console.WriteLine($"  Selection Mark {i} is {selectionMark.State}.");
        Console.WriteLine($"    Its bounding box is:");
        Console.WriteLine($"      Upper left => X: {selectionMark.BoundingPolygon[0].X}, Y= {selectionMark.BoundingPolygon[0].Y}");
        Console.WriteLine($"      Upper right => X: {selectionMark.BoundingPolygon[1].X}, Y= {selectionMark.BoundingPolygon[1].Y}");
        Console.WriteLine($"      Lower right => X: {selectionMark.BoundingPolygon[2].X}, Y= {selectionMark.BoundingPolygon[2].Y}");
        Console.WriteLine($"      Lower left => X: {selectionMark.BoundingPolygon[3].X}, Y= {selectionMark.BoundingPolygon[3].Y}");
    }
}

foreach (DocumentStyle style in result.Styles)
{
    // Check the style and style confidence to see if text is handwritten.
    // Note that value '0.8' is used as an example.

    bool isHandwritten = style.IsHandwritten.HasValue && style.IsHandwritten == true;

    if (isHandwritten && style.Confidence > 0.8)
    {
        Console.WriteLine($"Handwritten content found:");

        foreach (DocumentSpan span in style.Spans)
        {
            Console.WriteLine($"  Content: {result.Content.Substring(span.Index, span.Length)}");
        }
    }
}

Console.WriteLine("The following tables were extracted:");

for (int i = 0; i < result.Tables.Count; i++)
{
    DocumentTable table = result.Tables[i];
    Console.WriteLine($"  Table {i} has {table.RowCount} rows and {table.ColumnCount} columns.");

    foreach (DocumentTableCell cell in table.Cells)
    {
        Console.WriteLine($"    Cell ({cell.RowIndex}, {cell.ColumnIndex}) has kind '{cell.Kind}' and content: '{cell.Content}'.");
    }
}
```

W celu zweryfikowania aplikacji posłużono się dokumentem pdf udostępnionym w internecie pod adresem: https://www.w3.org/WAI/WCAG21/working-examples/pdf-table/table.pdf
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/dd15f482-e62e-4fd4-846f-e6ba661df0fb)

Program znalazł pary klucz-wartość. Kilka pierwszych:
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/9f0634a0-a2ab-4a03-906e-d34e266e0d94)

Program wykrył również tabelę i jej zawartość i przedstawił co znajduje się w której komórce:
![image](https://github.com/WojciechZ04/PUCH_lab/assets/120134082/31b51134-ce35-40d2-8f35-4235b3f5b728)

Niestety nie udało mi się odczytać plików z dysku lokalnego oraz udostępnionych na Google Drive.
