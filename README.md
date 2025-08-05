# Contribution Guidelines

These are my personal contribution guidelines and do not represent that of any company.

None of these are strict rules. Some are beliefs, some are guidelines. Some are big, some are small. Some depend on pre-existing project rules, some depend on pre-existing team rules and culture. They can be broken and adapted depending on the specific circumstances, and they can evolve over time.

- Start simple.
- Be pragmatic. Don’t add a layer of abstraction just because some piece of literature said so.
- Keep it at one commit per PR. There could be exceptions as long as the commits are meaningfully split up. For example, the first commit introduces a regression test that reproduces a problem and shows the current unintended behavior of the application. The second commit fixes the problem and shows the change in behavior in the test.
- Always `rebase`. Avoid `merge`.
- RUG over DRY. Repeat Until Good. Don't blindly follow DRY for DRY's sake. If repetition makes the logic easier to read and the problem easier to reason about, then repeat yourself until it makes sense to make it generic. For example, something like this *could* be generalized into its own logic, but this is good and straightfoward enough. If a few more parameters need to be handled, maybe that would be a stronger case for generalizing the logic. In the meantime, repetition is *okay*.

    ```python
    def __init__(self, key: str = None, secret: str = None):
      if key:
        self._key = key
      elif os.environ.get("APPLICATION_KEY", None):
        self._key = os.environ["APPLICATION_KEY"]
      else:
        raise ValueError("Key needs to be configured")

      if secret:
        self._secret = secret
      elif os.environ.get("APPLICATION_SECRET", None):
        self._secret = os.environ["APPLICATION_SECRET"]
      else:
        raise ValueError("Secret needs to be configured")
    ```

- Have a max line length set (I use 100) for business logic. Having a max line length makes side by side editing, reading, and reviewing easier.
- Don’t introduce new variables or references unless needed in multiple places or it makes the 100 column limit easier to implement. The less references the reader (future me probably) has to keep in their mind when reading the code, the faster it is to understand what is happening. For example, don't do this:

    ```ruby
    USERS_TIMEOUT_IN_SECONDS = 10
    # ... Somewhere in the file 239 lines down.
    client.get_users(timeout_in_seconds: USERS_TIMEOUT_IN_SECONDS)
    # ... `USERS_TIMEOUT_IN_SECONDS` never used again, anywhere, not even tests.
    ```

    the kwarg `timeout_in_seconds` clearly conveys what we're talking about here. This is perfectly fine despite the common convention against inline constants:

    ```ruby
    client.get_users(timeout_in_seconds: 10)
    ```

    Introduce the reference to `10` only when/if some other part of the code needs the same value or you already know you're absolutely going to use it soon somewhere else.
- Prefer higher level testing with as little mocking as possible over unit testing:
  - Higher level testing is harder to set up and slower to run, but these tests are more realistic, exercise significantly more lines of the codebase, and are more effective than unit tests that target a single function and mock everything around that function.
  - They allow you to refactor with confidence and speed because you don’t have to change lots small unit tests after every single business logic refactor change. [This guy](https://www.youtube.com/watch?v=LcJKxPXYudE&t=24m25s) said it better.
  - They encourage you to think about the higher level impact of changes. “What would the http request/inputs to the program/environment have to look like for the logic in this function to run and what would its impact to the final output be?”
  - They closer resemble how clients would actually use and test for correctness of the program/service.
