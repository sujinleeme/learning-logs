# Alchemist Camp - Elixir Guessing Game

[Alchemist Camp](https://alchemist.camp/) - [Lesson 1: The guessing game](https://alchemist.camp/episodes/guessing-game
)

## Exercise

Guess between a low number and a high -> guess middle number

Tell user our guess

  "yes" -> game over

  "bigger" -> bigger(low, high)
  
  "smaller" -> smaller(low, high)
  
  anything else -> tell user to enter a valid response

Challenge: See if you can make a program that asks the user's name and then greets them by name... and has a special response for users who enter your name.

### Example

Start with `GuessingGame.main()`.

```elixir
defmodule GuessingGame do
  @user_default_name "Anonymous"

  def main() do
    user_name = greeting()
    start(user_name)
  end

  def greeting() do
    user_input_name = read_user_name()

    name =
      if String.length(user_input_name) > 0 do
        user_input_name
      else
        @user_default_name
      end

    IO.puts("Hello, #{name}!")
    name
  end

  def start(name) do
    start = IO.gets("Do you want to start game?\n")

    case String.trim(start) do
      "yes" ->
        first_user_guess = IO.gets("Enter your first number\n") |> read_user_guess_number()

        second_user_guess = IO.gets("Enter your second number\n") |> read_user_guess_number()

        guess(first_user_guess, second_user_guess)

      "no" ->
        "Bye, #{name}. See you again. :)"

      _ ->
        IO.puts(~s{Type "yes" or "no"})
        start(name)
    end
  end

  def read_user_name() do
    IO.gets("Hello. What's your name?\n") |> String.trim()
  end

  def read_user_guess_number(num) do
    String.trim(num) |> String.to_integer()
  end

  def guess(a, b) when a > b, do: guess(b, a)

  def guess(low, high) do
    answer = IO.gets("Hmm... maybe you're thinking of #{mid(low, high)}?\n")

    case String.trim(answer) do
      "bigger" ->
        bigger(low, high)

      "smaller" ->
        smaller(low, high)

      "yes" ->
        "I knew I could guess your number!"

      _ ->
        IO.puts(~s{Type "bigger", "smaller" or "yes"})
        guess(low, high)
    end
  end

  def mid(low, high) do
    div(low + high, 2)
  end

  def bigger(low, high) do
    new_low = min(high, mid(low, high) + 1)
    guess(new_low, high)
  end

  def smaller(low, high) do
    new_high = max(low, mid(low, high) - 1)
    guess(low, new_high)
  end
end
```

`greeter.ex`

```elixir
defmodule Greeter do
  @author "Mark"
  def start do
    name = IO.gets("Hi, there! What's your name\n") |> String.trim()

    if name == @author do
      "Wow! #{@author} is my favorite name. I was programmed by saome named #{@author}!"
    else
      "Hi, #{name}. It's nice to meet you."
    end
  end
end
```
