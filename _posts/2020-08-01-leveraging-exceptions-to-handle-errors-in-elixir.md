---
layout: post
title: Leveraging Exceptions to handle errors in Elixir
date: 2020-08-01
background: '/img/posts/exception.jpg'
---

Returning a tagged tuple `{:ok, result} | {:error, reason}` is the de facto practice to handle errors in Elixir, but that may not be enough for all situations and this article will explore how to leverage [Exceptions](https://hexdocs.pm/elixir/Exception.html) to enrich errors and the benefits of doing so.

Looks like that subject is a recurring source of discussion, as [Martin Gausby](https://twitter.com/gausby) [asked](https://twitter.com/gausby/status/1283017670368145408) the community how to deal with expected and unexpected errors, and turns out [Michał Muskała](https://twitter.com/michalmuskala) has already [introduced](https://web.archive.org/web/20180414015950/http://michal.muskala.eu/2017/02/10/error-handling-in-elixir-libraries.html#comments-whatyouhide-errors) a clever technique to handle errors that was used by [Andrea Leopardi](https://twitter.com/whatyouhide) on libraries [Mint](https://github.com/elixir-mint/mint/blob/9b19e09994d85b9c3af05279340cefc6c440ea67/lib/mint/http_error.ex) and [Redix](https://github.com/whatyouhide/redix/blob/947a5d6f68a4b8ad0165c6b88b221a42964a5f50/lib/redix/exceptions.ex). He [tweeted defending](https://twitter.com/whatyouhide/status/1266405013460594695) that exceptions are a great return value, so let's dig in to find out how that works.

#### TLDR

* Encapsulate all possible errors of a function into a custom Exception;
* Leverage Exceptions message mechanism to avoid coupling;
* Let the caller decide if the error is expected or not;

#### Code example:

**Error exception** _lib/my_app/error.ex_

```elixir
defmodule MyApp.Error do
  @type t() :: %__MODULE__{
          module: module(),
          reason: atom(),
          changeset: Ecto.Changeset.t() | nil
        }

  defexception [:module, :reason, :changeset]

  @spec wrap(module(), atom()) :: t()
  def wrap(module, reason), do: %__MODULE__{module: module, reason: reason}

  @spec wrap(module(), atom(), Ecto.Changeset.t()) :: t()
  def wrap(module, reason, changeset) do
    %__MODULE__{module: module, reason: reason, changeset: changeset}
  end

  @doc """
  Return the message for the given error.

  ### Examples

       iex> {:error, %MyApp.Error{} = error} = do_something()
       iex> Exception.message(error)
       "Unable to perform this action."

  """
  @spec message(t()) :: String.t()
  def message(%__MODULE__{reason: reason, module: module}) do
    module.format_error(reason)
  end
end
```

**Context** _lib/my_app/accounts.ex_

```elixir
defmodule MyApp.Accounts do
  @spec register(map()) :: {:ok, User.t()} | {:error, MyApp.Error.t()}
  def register(attrs) do
    # simulate a function that may return more than one type of error
    case has_permission?(attrs) do
      true ->
        # simulate that something wrong happened on register,
        # and note that changeset is just a regular changeset
        MyApp.Error.wrap(__MODULE__, :register, changeset)
        
      false ->
        # another situation requires another type of error
        MyApp.Error.wrap(__MODULE__, :insufficient_permisions)
    end 
    
    # and other errors could happen...
  end
  
  # translate the error case into a friendly message
  def format_error(:register), do: "Unable to register account."
  def format_error(:insufficient_permissions), do: "Unable to perform action due to insufficient permissions."
end
```

**Caller** _LiveView, Controller, etc_

```elixir
case MyApp.Accounts.register(attrs) do
  {:ok, user} ->
    continue_happy_path()
  
  {:error, error} ->
    socket =
      socket
      |> put_flash(:error, Exception.message(error))
      |> assign(:changeset, error.changeset)
    
    {:noreply, socket}
end
```

You may be asking, why not just return `{:error, :reason}` or even `{:error, "message"}`? First of all, try to avoid returning a string because that will complicate the pattern matching and a simple change will break your system, on the other hand returning an atom is totally fine when your function doesn't need to deal with different errors and messages. But usually context or complex functions has more outcomes than just a single possible error, and besides that they're usually consumed by another layer that needs to transform that error into useful feedback for the user.

#### Some practical scenarios and benefits

* In LiveViews or Controllers, you usually need to display a flash message to let the user know what's happening. If you return `{:error, atom()}` that means you need to pattern match all possible atoms to create the proper message. With exceptions, all you need to do is to implement `format_error` in a single place and call `Exception.message(error)`.
* If that same function changes current atom or adds another return value, you have to go through all places where that function is called to update the pattern match. That's error-prone as you may miss something and the compiler won't help here
* Shared and reused messages. Think about generic errors such as authorization, system errors, and others. Those errors produce the same message everywhere, which requires duplicating the pattern match and the message definition in different places. That can be solved by relying on the expcetion message mechanism.
* Cohesion. Implementing a `format_message/1` close to where the error happens will improve maintainability.
* Transparency. By having the module and a reason in the error struct, you'll have the origin of that error, from where it's coming from. Stack traces won't help when the error is expected and it's not raised.

#### But what about that changeset in the middle of the error?

Regular apps, especially web apps, depends a lot on [Changesets](https://hexdocs.pm/ecto/Ecto.Changeset.html) to return feedback to users but also has to deal with complementary errors on more complex scenarios where returning just an invalid changeset isn't enough. Suppose a function that deals with form submission but also has to call an external service, check permissions, or deal with that crazy legacy rule. Many different errors or situations may happen: the changeset may be invalid, an external service may be offline, or maybe an error happened but the changeset is valid and needs to be updated to reflect changes on the template. That would require more complex return values like adding more values on the tagged tuple, a struct to store all values, or something. A better approach is to return either `{:error, %MyApp.Error{reason: :invalid_input, changeset: changeset}}`, `{:error, %MyApp.Error{reason: :billing_service_offline}}`, or whatever is needed. All you need will be encapsulated on the exception struct. With that return, you can either display inline errors on form if the changeset is valid, call `Exception.message(error)` to give proper feedback for the user or update the changeset while giving feedback for the user about another error. It's very flexible and simple.

#### Error is part of your application

You're not limited to a generic `%MyApp.Error{reason: atom()}`, in fact you can implement explicit errors like `%MyApp.OfflineService{reason: atom(), status: integer()}` instead of `%MyApp.Error{reason: :billing_service_offline}` to enrich errors specific to your app's domain. Some benefits include leveraging pattern matching for control flow, explicit errors when raising or reading your code, and encapsulate metadata about specific errors, but not limited to those benefits.

#### Error is expected or not?

That depends on the caller because that usually is tied to the current situation. Suppose a function that calculates something based on a set of data, if that function is called by a Controller or LiveView probably the user is waiting for feedback, but if that function is called by an async process (mostly a background process) there's no reason to present a message so raising to force the process to restart may be the best approach. In short, let the caller decide:

* If the error is expected or needs to display feedback, call `Exception.message(error)`
* If the error is not expected or there's no way to recover from that, call `raise error`

Remember that error is in fact an exception, so raising it is simple as calling `raise error`, but remember to [avoid using try/rescue](https://elixir-lang.org/getting-started/try-catch-and-rescue.html#errors) for control flow and reserve that for situations where the function has reached the end of the line and there's nothing else to do unless raising.

##### Notes

Thanks to:
* [Andrea Leopardi](https://twitter.com/whatyouhide) for reviewing this article and also for giving the inspiration to write it.
* [Michał Muskała](https://twitter.com/michalmuskala) for introducing this technique.
* [Eduardo Hernandes](https://twitter.com/eduardodeoh) for giving the change to pair program and experiment.
