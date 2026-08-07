---
url: https://sebnilsson.com/blog/csharp-datetimeoffset-formats-iso-8601-rfc-3339-json-and-unix-time
date_fetched: 2026-08-07
---

Format and parse C# DateTimeOffset as ISO 8601, RFC 3339, JSON and Unix time, and see which formats survive the round trip between services.

Date and time become harder to work with as soon as a timestamp leaves your single-language application. In JSON, query strings, headers, logs, and file names, it becomes text or a number that another system has to interpret correctly.

In this article, we'll go through the practical formats for services, TypeScript/JavaScript frontends, logs, and files. For each one, we'll look at how to produce it from a `DateTimeOffset` and what survives when you parse it back.

In modern .NET code, use `DateTimeOffset` for timestamps. **Unlike  DateTime, it stores both the clock time and its UTC offset**, which is the information most easily lost at a service boundary.

Every example below formats and parses the same timestamp, a summer afternoon in Central European Summer Time:

```
var timestamp = new DateTimeOffset(
    year: 2026, month: 7, day: 14,
    hour: 9, minute: 11, second: 30, millisecond: 123,
    offset: TimeSpan.FromHours(2));
```
This overview shows what each format preserves when parsed back:

| Format | Example output | Parses back to `DateTimeOffset` | 
|---|---|---|
| ISO 8601 / RFC 3339 with offset | `2026-07-14T09:11:30.123+02:00` | Yes, exactly | 
| RFC 3339 in UTC | `2026-07-14T07:11:30.123Z` | Yes, as UTC | 
| `System.Text.Json`default | `2026-07-14T09:11:30.123+02:00` | Yes, exactly | 
| .NET round-trip ( `O`) | `2026-07-14T09:11:30.1230000+02:00` | Yes, exactly | 
| Unix seconds | `1784013090` | Yes, as UTC, without ms | 
| Unix milliseconds | `1784013090123` | Yes, as UTC | 
| RFC 1123 ( `R`) | `Tue, 14 Jul 2026 07:11:30 GMT` | Yes, as UTC, without ms | 
| Sortable ( `s`) | `2026-07-14T09:11:30` | No, offset missing | 
| Universal sortable ( `u`) | `2026-07-14 07:11:30Z` | Yes, as UTC, without ms | 
| Compact UTC | `20260714T071130.123Z` | Yes, as UTC | 
| Date only | `2026-07-14` | No, time and offset missing | 
| Time only | `09:11:30.123` | No, date and offset missing | 
| Localized display | `Tuesday, July 14, 2026 9:11:30 AM` | No, offset and ms missing | 

One of the first four rows is almost always the right answer for an API. Of the .NET standard specifiers, `O` preserves the offset and all seven fractional digits, `R` is useful for HTTP headers, `s` drops the offset, and `u` converts to UTC. The remaining formats deliberately discard information, which is useful when you intend it, but it's a bug when you don't.

UTC (Coordinated Universal Time) is the global reference time represented by the zero offset `+00:00`, often written as `Z`. **Converting a timestamp to UTC preserves the exact instant and its available fractional precision in one unambiguous value, which makes it a reliable choice for storage and exchange between systems.**

ISO 8601 is a large standard that covers dates, times, UTC, local time with an offset, durations, intervals, and more. It's broad enough that "we use ISO 8601" isn't really a contract on its own. For timestamps exchanged between applications, RFC 3339 defines a much narrower profile of it, and that profile is what most APIs mean when they say ISO 8601.

**If you have no other requirement, an RFC 3339 timestamp with an explicit offset is the format to send.** It's unambiguous, and almost every language and framework can parse it. RFC 3339 timestamps also sort chronologically as text when they use the same offset representation and the same number of fractional digits.

Use one format for an explicit offset and another when the contract requires UTC with `Z`:

```
const string offsetFormat = "yyyy-MM-dd'T'HH:mm:ss.fffK";
const string utcFormat = "yyyy-MM-dd'T'HH:mm:ss.fff'Z'";
var withOffset = timestamp.ToString(offsetFormat, CultureInfo.InvariantCulture);
// 2026-07-14T09:11:30.123+02:00
var inUtc = timestamp.ToUniversalTime().ToString(utcFormat, CultureInfo.InvariantCulture);
// 2026-07-14T07:11:30.123Z
```
Lowercase `fff` requires exactly three fractional digits. Use `FFFFFFF` instead when trailing zeros and the decimal point may be omitted. For a `DateTimeOffset`, the `K` token is equivalent to `zzz`: both write the numeric offset, including `+00:00` for UTC.

**Only append  Z after converting the clock time to UTC.** Adding it directly to the example's 

`09:11:30` moves the timestamp two hours into the future without raising an error.The offset format parses directly. Since the `Z` in `utcFormat` is a literal, the UTC format needs explicit parsing styles:

```
var utcStyles = DateTimeStyles.AssumeUniversal | DateTimeStyles.AdjustToUniversal;
var parsedOffset = DateTimeOffset.ParseExact(
    withOffset, offsetFormat, CultureInfo.InvariantCulture, DateTimeStyles.None);
// 2026-07-14T09:11:30.1230000+02:00
var parsedUtc = DateTimeOffset.ParseExact(
    inUtc, utcFormat, CultureInfo.InvariantCulture, utcStyles);
// 2026-07-14T07:11:30.1230000+00:00
```
The offset says how far the clock was from UTC, not which time zone it was in. If the receiver needs the zone itself, send the IANA time zone ID separately. The `DateTimeStyles` documentation covers the UTC parsing flags used again below.

ISO 8601 also has a basic format without separators, which works where `:` isn't allowed or is inconvenient, such as file names, log identifiers, cache keys, and exported data. Fixed-width UTC values sort chronologically as plain text, which is often the real reason to use them:

```
const string fileFormat = "yyyyMMdd'T'HHmmss.fff'Z'";
var fileTimestamp = timestamp
    .ToUniversalTime()
    .ToString(fileFormat, CultureInfo.InvariantCulture);
// 20260714T071130.123Z
DateTimeOffset.ParseExact(fileTimestamp, fileFormat, CultureInfo.InvariantCulture, utcStyles);
// 2026-07-14T07:11:30.1230000+00:00
```
`DateTimeOffset.Parse` doesn't accept this basic format, and it has no format parameter. Use `DateTimeOffset.ParseExact` with `fileFormat` and keep the format string next to the code that reads these values back. **Keep the field widths and the UTC suffix consistent if you rely on alphabetical sorting**, since a single variable-width field breaks the ordering for every file in the directory.

When a `DateTimeOffset` is part of a JSON request or response, you usually shouldn't call `ToString` at all. `System.Text.Json` reads and writes the extended ISO 8601 profile by default, which is also valid RFC 3339, and it's what ASP.NET Core uses for minimal APIs and controllers out of the box:

```
var json = JsonSerializer.Serialize(timestamp);
// "2026-07-14T09:11:30.123+02:00"
var parsed = JsonSerializer.Deserialize<DateTimeOffset>(json);
// 2026-07-14T09:11:30.1230000+02:00
```
The default output trims trailing zeros, so it writes a timestamp with whole seconds as `2026-07-14T09:11:30+02:00`. `System.Text.Json` writes a `DateTimeOffset` in UTC with `+00:00`, but a `DateTime` with `DateTimeKind.Utc` with `Z`:

```
JsonSerializer.Serialize(timestamp.ToUniversalTime()); // "...T07:11:30.123+00:00"
JsonSerializer.Serialize(timestamp.UtcDateTime);       // "...T07:11:30.123Z"
```
Both are correct RFC 3339 and both parse the same way in .NET and in TypeScript/JavaScript. It only becomes a problem when something on the other side compares timestamp strings for equality, or when a test asserts on the exact text.

If the API contract requires UTC with `Z` and a fixed precision, add a converter once instead of formatting values by hand throughout the application:

```
public sealed class UtcDateTimeOffsetJsonConverter : JsonConverter<DateTimeOffset>
{
    private const string Format = "yyyy-MM-dd'T'HH:mm:ss.fff'Z'";
    public override DateTimeOffset Read(
        ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)
    {
        return DateTimeOffset.ParseExact(
            reader.GetString()!,
            Format,
            CultureInfo.InvariantCulture,
            DateTimeStyles.AssumeUniversal | DateTimeStyles.AdjustToUniversal);
    }
    public override void Write(
        Utf8JsonWriter writer, DateTimeOffset value, JsonSerializerOptions options)
    {
        writer.WriteStringValue(
            value.ToUniversalTime().ToString(Format, CultureInfo.InvariantCulture));
    }
}
```
Because `Read` uses `ParseExact`, this converter accepts only UTC timestamps that end in `Z` and contain exactly three fractional digits. The default `System.Text.Json` `DateTimeOffset` format uses variable fractional precision and writes UTC as `+00:00`, so every client has to follow this custom contract too.

Register it once with `options.Converters.Add(new UtcDateTimeOffsetJsonConverter())`. **Let the serializer own the wire format, and let a custom converter own any deviation from it.**

A browser can read either the default JSON string or Unix milliseconds as the same instant:

```
const fromText = new Date("2026-07-14T09:11:30.123+02:00");
const fromUnix = new Date(1784013090123);
console.log(fromText.toISOString());                    // 2026-07-14T07:11:30.123Z
console.log(fromText.getTime());                        // 1784013090123
console.log(fromText.getTime() === fromUnix.getTime()); // true
```
A TypeScript/JavaScript `Date` stores milliseconds since the epoch, so it uses `+02:00` to find the instant and then discards the offset. `JSON.stringify` writes the result in UTC with `Z`. **If the frontend needs the sender's original offset, send it separately.**

There are three traps worth knowing about at this boundary:

`new Date("2026-07-14T09:11:30")` means something different for every visitor, which is exactly what the `s` format produces.`new Date("2026-07-14")` is midnight UTC, so the same string is inconsistent with the rule above. This is the classic reason a date shows up as the day before for users west of UTC.`new Date` land in January 1970.`new Date(1784013090)` is `1970-01-21T15:33:33.090Z`, because the constructor expects milliseconds.The ECMAScript Date Time String Format defines exactly three fractional digits. The seven digits produced by `O` therefore rely on engine-specific fallback parsing. **Keep  O for .NET-to-.NET text rather than a browser-facing wire format.**

Unix time, also called epoch time, counts from `1970-01-01T00:00:00Z` and is useful when a contract expects a number. `DateTimeOffset` has built-in Unix time support in both directions:

```
var seconds = timestamp.ToUnixTimeSeconds();           // 1784013090
var milliseconds = timestamp.ToUnixTimeMilliseconds(); // 1784013090123
DateTimeOffset.FromUnixTimeSeconds(seconds);
// 2026-07-14T07:11:30.0000000+00:00
DateTimeOffset.FromUnixTimeMilliseconds(milliseconds);
// 2026-07-14T07:11:30.1230000+00:00
```
The original offset is gone when the value comes back as UTC. Seconds discard the fractional second, while milliseconds discard any finer 100-nanosecond ticks. **Agree on the unit, because reading seconds as milliseconds produces a date in January 1970 rather than an error.** JWT claims use seconds, while JavaScript `Date` uses milliseconds.

Sometimes the value is not a timestamp. Use `DateOnly` for birthdays and billing days, and `TimeOnly` for opening times and daily alarms:

```
var dateAtOffset = DateOnly.FromDateTime(timestamp.DateTime);    // 2026-07-14
var timeAtOffset = TimeOnly.FromDateTime(timestamp.DateTime);    // 09:11:30.123
JsonSerializer.Serialize(dateAtOffset); // "2026-07-14"
JsonSerializer.Serialize(timeAtOffset); // "09:11:30.1230000"
```
For display, standard formats such as `d` and `F` use the supplied `CultureInfo`:

```
var culture = CultureInfo.GetCultureInfo("sv-SE");
timestamp.ToString("d", culture); // 2026-07-14
timestamp.ToString("F", culture); // tisdag 14 juli 2026 09:11:30
```
**Choose the relevant offset before extracting a calendar value, and keep display text as output only.** A date near midnight may differ between the original offset and UTC. Culture changes the representation, not the time zone, which is a separate conversion using `TimeZoneInfo`. For a browser-side example using older tools, see Display Local DateTime with Moment.js in ASP.NET.

When an API accepts more than one representation, `TryParseExact` validates the input against an array of formats and rejects everything else:

```
string[] acceptedFormats =
{
    "O",
    "yyyy-MM-dd'T'HH:mm:ss.fffK",
    "yyyy-MM-dd'T'HH:mm:ss.FFFFFFFK"
};
if (DateTimeOffset.TryParseExact(
    input, acceptedFormats, CultureInfo.InvariantCulture, DateTimeStyles.None, out var parsed))
{
    // 2026-07-14T09:11:30.1230000+02:00
}
```
The more flexible `DateTimeOffset.TryParse` handles a much wider range of input, which is convenient until it accepts something you did not intend. **When the input has no offset,  TryParse fills in the offset of the machine that runs it**, so the same request parses differently on a developer's laptop and on a server in another region. Pass 

`DateTimeStyles.AssumeUniversal` when a missing offset must mean UTC, and use `TryParseExact` when the accepted formats are part of the contract.Keep timestamps as `DateTimeOffset` inside the application and database, and format them only at system boundaries. SQL Server has a native `datetimeoffset` type, while strings lose validation, date arithmetic, and reliable sorting across offsets.

`System.Text.Json` default, or a documented RFC 3339 contract. Keep `O` for .NET-to-.NET text that needs all seven fractional digits.**Pick a format based on the offset and precision it preserves, parse the same format you write, and supply any missing offset explicitly.** A wrong timestamp is still valid, which is why these mistakes rarely raise an error.

*For the original  DateTime and .NET Framework implementation, see the original article.*
