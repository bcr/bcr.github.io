---
layout: post
title:  Stupid JSON Formatting Tricks in C#
date:   2022-10-14 21:06:22 -0700
author: Blake Ramsdell
categories: coding
tags: c# json
---
I started messing with [PrettyPrompt](https://github.com/waf/PrettyPrompt). I
figured out how to generate syntax highlighting for JSON. Here it is.

1. PrettyPrompt has a FormattedString concept. You give it an array of
   FormatSpan with start and offset and color along with a string.
2. System.Text.Json has a
   [Utf8JsonReader](https://learn.microsoft.com/en-us/dotnet/api/system.text.json.utf8jsonreader)
   concept where it tells you all the tokens.
3. For each token, determine the start and length, then apply a format.

```csharp
using PrettyPrompt.Highlighting;
using System.Text;
using System.Text.Json;

namespace Bcr.BluesWireless.Notecard.Console;

public class JsonHelper
{
    static Dictionary<JsonTokenType, AnsiColor> _styleDictionary = new()  {
        { JsonTokenType.String, AnsiColor.BrightGreen },
        { JsonTokenType.Number, AnsiColor.Yellow },
        { JsonTokenType.True, AnsiColor.Yellow },
        { JsonTokenType.False, AnsiColor.Yellow },
    };

    public static FormattedString GetFormattedJson(string json)
    {
        var reader = new Utf8JsonReader(Encoding.UTF8.GetBytes(json));
        var formatting = new List<FormatSpan>();

        while (reader.Read())
        {
            int start = (int) reader.TokenStartIndex;
            int length = reader.HasValueSequence ? (int) reader.ValueSequence.Length : reader.ValueSpan.Length;

            switch (reader.TokenType)
            {
                case JsonTokenType.PropertyName:
                case JsonTokenType.String:
                    length += 2;
                    break;
            }

            if (_styleDictionary.ContainsKey(reader.TokenType))
            {
                formatting.Add(new FormatSpan(start, length, _styleDictionary[reader.TokenType]));
            }
        }

        return new FormattedString(json, formatting);
    }
}
```
* Update `_styleDictionary` however you want. I'll probably do `null` at some
  point and I won't include it here, so there.
* There is a hack for `PropertyName` and `String` -- they point to the opening
  quote, and the length is the contents between the quotes. So I add two.
