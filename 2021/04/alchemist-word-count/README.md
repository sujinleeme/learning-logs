# Alchemist Camp - Word Count

[Alchemist Camp](https://alchemist.camp/) - [Lesson 2: Word Count](https://alchemist.camp/episodes/word-count)

## Exercise

- Use `IO.gets` to ask user the name of a file to open
- Use `String.trim` to get rid of trailing enter they press
- Split the string into words with `String.split` and a regular expression
- Filter out non words with `Enum.filter`
- Count the words with `Enum.count`

```elixir
filename = IO.gets("File to count the words from: ") |> String.trim()

words =
  File.read!(filename)
  |> String.split(~r{(\\n|[^\w'])+})
  |> Enum.filter(fn x -> x != "" end)

words |> Enum.count() |> IO.puts()
```

## Challenge

User can optionally count the number of lines or characters a file instead of the number of words.

```elixir
filename =
  IO.gets("File to count the words from (h for help):\n")
  |> String.trim

if filename == "h" do
  IO.puts """
Usage: [filename] -[flags]
Flags
-l     displays line count
-c     displays character count
-w     displays word count (default)
Multiple flags may be used. Example usage to display line and character count:
somefile.txt -lc
"""
else
  parts = String.split(filename, "-")
  filename = List.first(parts) |> String.trim
  flags = case Enum.at(parts, 1) do
    # set only "w" flag if none were set
    nil     ->  ["w"]
    chars   ->  String.split(chars, "") |> Enum.filter(fn x -> x != "" end)
  end

  body = File.read! filename
  lines = String.split(body, ~r{(\r\n|\n|\r)})
  words =
    String.split(body, ~r{(\\n|[^\w'])+})
    |> Enum.filter(fn x -> x != "" end)
  chars = String.split(body, "") |> Enum.filter(fn x -> x != "" end)

  Enum.each(flags, fn flag ->
    case flag do
      "l"     ->  IO.puts "Lines: #{Enum.count(lines)}"
      "w"     ->  IO.puts "Words: #{Enum.count(words)}"
      "c"     ->  IO.puts "Chars: #{Enum.count(chars)}"
      _       ->  nil
    end
  end)
end
```
