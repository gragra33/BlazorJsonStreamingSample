# Blazor WASM Stream Decoding of Large JSON files

This sample application uses the [Utf8JsonAsyncStreamReader](https://www.nuget.org/packages/Utf8JsonAsyncStreamReader) as an asynchronous forward-only streaming JSON parser and deserializer (based on System.Text.Json.Utf8JsonReader) to handle over-sized JSON files.

All that is required is to open a stream:
```cs
Stream jsonStream = await httpClient.GetStreamAsync(url);
using Utf8JsonAsyncStreamReader reader = new(jsonStream);

// move to the start of the array and read the stream
while (await reader.ReadAsync())
    await ProcessJsonStreamAsync(reader);
```
Then parse and process the stream:
```cs
async Task ProcessJsonStreamAsync(Utf8JsonAsyncStreamReader reader)
{
    if (reader.TokenType != JsonTokenType.PropertyName)
        return;

    // process properties for data that we want
    if (reader.GetString() == "colors")
    {
        // move to the start of the array
        await reader.ReadAsync();

        // step through each complete object in the Json Array
        while (await reader.ReadAsync() && reader.TokenType != JsonTokenType.EndArray)
        {
            // convert the json object to a class
            await ProcessObjectAsync(reader);

            // report progress every 25 objects...
            if (ColorNames.Count % 25 != 0 || ColorNames.Count == 0)
                continue;

            // update UI to show current count progress
            await InvokeAsync(StateHasChanged);
            
            // yield to give the UI time to update - do not use Task.Yield()
            await Task.Delay(1);
        }
    }
}

async Task ProcessObjectAsync(Utf8JsonAsyncStreamReader reader)
{
    Color? color = await reader.DeserializeAsync<Color>(jsonSerializerOptions);

    // process color...
    if (color is null)
        return;

    ColorNames.Add($"{color.name}: {color.hex}");
}
```
`Utf8JsonAsyncStreamReader` does the rest of the work for you.

**NOTE:** `Utf8JsonAsyncStreamReader` can be used in any .Net (Core) project, not just Blazor!