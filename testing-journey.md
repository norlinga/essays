# A First Step in a Testing Journey

I'm writing this essay to my past, testing-skeptical self.  This is the essay I wish I had read at the beginning of my Ruby career.

> _Dear Past Self,_
> 
> _Trust me - I understand.  You had every reason to be annoyed by the perpetually broken test suites and the testing acolytes who couldn't explain > themselves.  All you saw around you were wild eyed fanatics who spent their time arguing about matchers and tuning Rspec outputs to have better > colors... but couldn't explain what was wrong when anything broke in production.  You "tested" in IRB and you shipped code.  And you shipped it fast!  > Which is not to say that you were living a stress-free life.  New features were scary, and building new features was slow and getting slower all the > time.  Not great._
> 
> _And yet here I am, reconciled and at peace with testing.  Read on.  Trust.  And know that your journey will be worth it.  Take that first step..._
> 
> _Sincerely,_
> 
> _Future You_

**A quick aside to my Dear Reader**: are you just starting your journey in Ruby?
Awesome!  And welcome!!!  You're going to love it here.
Reading this essay, you'll get a glimpse into my rational for pushing testing early and often in development.

Or maybe you're a reasonably seasoned Ruby practitioner who's disillusioned with tests?
Maybe you never got comfortable with testing in the first place?
I hope to help you break down some of your mental and emotional blocks.
Trust me, I carried some of those for years.

OR, you might be a very seasoned, committed Ruby tester and you don't know why you get pushback when you talk dreamily of testing.
There's chinks in the armor of Ruby testing resistence, and I hope to help you exploit those weaknesses and speak effectively about testing.

## The Ruby Testing TL;DR

Here's the thrilling conclusion of this essay, up front.
Testing in Ruby is:

- easier than you imagined, and
- accrues bigger benefits to than you thought possible

A `puts` or **REPL** or whatever-else-driven workflow might seem fast and sufficient in the quality department to start.
Whatever the perceived benefits of a **primarily-non-test-driven development** process might be, the benefits are illusory. 
Time and experience will remove any doubts that a test-driven workflow in Ruby is substantially easier and faster than any alternative.
A test-driven workflow is a repeatable, reliable, confidence-inspiring way to create beautiful and sustainable Ruby code.

The rest of this essay will be justifying these claims.
I hope you're willing to take the first step and to be open to the argument.
Let's get started with a typical contrived example.

## Scenario: The REPL-based Workflow 

(no doubt with `puts` statements everywhere...)

Maybe this scenario sounds familiar:

Let's say you've been given an `Array` that contains `Hashes` of user data.
The data looks like this:

```rb
[
  { name: "Jane", status: "active", email: "jane@example.com" },
  { name: "Tom",  status: "inactive", email: "tom@example.com" },
  { name: "Sara", status: "active", email: "sara@example.com" }
]
```

The goal of this exercise is to form two groups of users out of the data - active and inactive.
Each group is going to get their own promo emails: a "Welcome Back!" email for the inactives and a "Customer Appreciation" email for the actives.
The goal is to get the above data to look like this:

```rb
{
  "active" => ["jane@example.com", "sara@example.com"],
  "inactive" => ["tom@example.com"]
}
```

With the data transformed into a Hash of Arrays, you can pass off to the Emailing Thingy and you're done!
This looks easy enough, and you quickly hammer out a piece of code to validate your ideas in IRB:

```rb
irb(main):001* users = [
irb(main):002*   { name: "Jane", status: "active", email: "jane@example.com" },
irb(main):003*   { name: "Tom",  status: "inactive", email: "tom@example.com" },
irb(main):004*   { name: "Sara", status: "active", email: "sara@example.com" }
irb(main):005> ]
=>
[{:name=>"Jane", :status=>"active", :email=>"jane@example.com"},
...
irb(main):006* class UserEmailGrouper
irb(main):007*   def initialize(users)
irb(main):008*     @users = users
irb(main):009*   end
irb(main):010*
irb(main):011*   def group_by_status
irb(main):012*     result = {}
irb(main):013*     @users.each do |user|
irb(main):014*       result[user[:status]] ||= []
irb(main):015*       result[user[:status]] << user[:email]
irb(main):016*     end
irb(main):017*     result
irb(main):018*   end
irb(main):019> end
=> :group_by_status
irb(main):020> UserEmailGrouper.new(users).group_by_status
=> {"active"=>["jane@example.com", "sara@example.com"], "inactive"=>["tom@example.com"]}
irb(main):021>
```

Or maybe you're a next level REPL user and you wrote your code elsewhere and included it:

```rb
require_relative 'user_email_grouper'
```

which would be marginally smarter.
Anyway, everyone knows it's not always this simple and it doesn't always go this smoothly, BUT you have something working and you're ready to move on!

## It's Never That Simple - And It's Never That Smooth

The first time this code is runs in production, you find out - somehow - there's Users in the system without email addresses.
How that happened exactly, no one's quite sure.
All you know is, your code doesn't cover for `nil` emails.  Darn.

Oh, and there's some duplicate email addresses as well.
Again, shocked befuddlement but there it is.
The decision is made: any users that share duplicate email addresses should be dropped from the list entirely.

And there's more!
Turns out there's **sometimes** a column in some CSV somewhere that tracks who opted out of emails.
It's not always there, but when it is you should probably respect that field if you want your email reputation to stay high so your emails stay out of the spam folder.

_and for no reason that anyone can understand, someone decided that emails need to go out in alphabetical order. whatever.  okay._

---

These new requirements might come in all at once, or they might come in over weeks or months.
Whatever the case, you'll be revisiting this code over and over again, and if you're living that IRB life you know you'll be rebuilding your test data set over and over.
Or maybe saving test data dumps locally for reference?
You'll stare at the same sample output, straining to see if your filtered data was really filtered after all.
And you'll wonder if you're introducing bugs along the way, which means you'll be manually looking for regressions while trying to ensure all the new requirements are satisfied.

Your first explosion of code was exciting, satisfying, and fun!
But that was then - you're not having fun any more.

And you're not learning anything new, either!
You suspect that the code you wrote that first time around could be improved.
Maybe it could even be elegant, but it's part of a brittle system now.
You'd like to mess around and try new things but last time you tried to use a `.select()` or an `.inject()` was a disaster and you're becoming shy of new things.

There's a better way, and it's closer and easier than you think.

## Testing is SO Easy To Set Up

For this example we're going to use Minitest, a part of the Ruby Standard Library.

> **ℹ️ Note:** Rspec is easy to set up but we're going to stick to Minitest.  It's easier, powerful, and very fast!

For the code that follows I'm going to redo the same example from above, but drive the development process through tests.
I need to process an Array of Hashes and return a Hash of Arrays.
Let's jump into a fresh directory (pretending that this work isn't part of a larger project) and do a little light housekeeping:

```sh
% mkdir user_email_grouper_spike && cd user_email_grouper_spike
% bundle init
% bundle add minitest minitest-reporters
% touch app.rb
```

In the CLI above, I:

- created a new directory,
- initialized `bundler` and added our only two required Rubygems, and
- made a single file to hold all our work.

> **ℹ️ Note:** I'll run the code I write through `bundle exec` when I'm ready.  The `minitest-reporters` gem isn't even strictly necessary but I include it here because tooling matters, and `-reporters` will gives me better looking output.

Now I'm ready to open my favorite text editor and write some code.

Here's the first ephiphany - I can put both my test and implementation code in the same file, and it's _just going to work_ thanks to `minitest/autorun`.

I'm going to bring the code down from above in just a second, but first I want to run a smoke test for my test-driven development setup.
In the `app.rb` file:

```rb
# ./app.rb

require 'minitest'
require 'minitest/autorun'
require 'minitest/reporters'

Minitest::Reporters.use!

class UserEmailGrouperTest < Minitest::Test
  def test_user_email_grouper
    assert true
  end
end

class UserEmailGrouper
end
```

Save, and then on the command line:

```sh
% bundle exec ruby app.rb
```

gives me the following welcome output:

```sh
Finished in 0.00133s
1 tests, 1 assertions, 0 failures, 0 errors, 0 skips
```

Success!

...But also: _so what?_

Your blood pressure might be rising right now.
You might start demanding to know, "What - exactly - does this prove?"
How is this an improvement on anything, at all?
This isn't just treading water, this is moving backward!
All this time wasted on setting up tests and there's no working code yet!!!

**True**.
But consider this - good tooling matters in real engineering, and I've **already** put all the tooling in place that I need.
The boilerplate is completely out of the way.
I've got the engine running.
From here onward, every test I write is pure momentum.
Trust.

## The Naive Example

Let’s walk quickly through the naive case — the "happy path" — so we can get to something more interesting.

Below is our first real test. You’ll see I’ve added the sample data, the expected output, and an assertion that ties them together:

```rb
# in the test class in ./app.rb

class UserEmailGrouperTest < Minitest::Test
  def test_returns_hash_with_email_arrays_as_values
    users = [
      { name: "Jane", status: "active", email: "jane@example.com" },
      { name: "Tom",  status: "inactive", email: "tom@example.com" },
      { name: "Sara", status: "active", email: "sara@example.com" }
    ]
    users_grouped_by_status = {
      "active" => ["jane@example.com", "sara@example.com"],
      "inactive" => ["tom@example.com"]
    }
    assert_equal users_grouped_by_status, UserEmailGrouper.new(users).group_by_status
  end
end
```

When I run the code again - now that the real test is in place - it blows up:

```sh
ERROR UserEmailGrouperTest#test_returns_hash_with_email_arrays_as_values
ArgumentError: wrong number of arguments (given 1, expected 0)
```

Since I have no real code, we'd expect it to fail.
I can fix this by wiring up a basic implementation using what I did in the console above.
The whole thing looks like this now:

```rb
# ./app.rb

require 'minitest'
require 'minitest/autorun'
require 'minitest/reporters'

Minitest::Reporters.use!

class UserEmailGrouperTest < Minitest::Test
  def test_returns_hash_with_email_arrays_as_values
    users = [
      { name: "Jane", status: "active", email: "jane@example.com" },
      { name: "Tom",  status: "inactive", email: "tom@example.com" },
      { name: "Sara", status: "active", email: "sara@example.com" }
    ]
    users_grouped_by_status = {
      "active" => ["jane@example.com", "sara@example.com"],
      "inactive" => ["tom@example.com"]
    }
    assert_equal users_grouped_by_status, UserEmailGrouper.new(users).group_by_status
  end
end

class UserEmailGrouper
  def initialize(users)
    @users = users
  end

  def group_by_status
    result = {}
    @users.each do |user|
      result[user[:status]] ||= []
      result[user[:status]] << user[:email]
    end
    result
  end
end
```

And we get the following results when we run our tests:

```sh
Finished in 0.00133s
1 tests, 1 assertions, 0 failures, 0 errors, 0 skips
```

Green!

This back and forth, "failure-to-success" is sometimes called **Red / Green** development.
The goal in red-green is to create a test case to drive feature development, then satisfy the test.
Before starting another feature in red-green development I would get the test suite to green, then start over with more red.

If I pedanticly red-green every feature in this essay... we're going to be here for a while.
Your job as the reader, from here on, is to presume that I'm doing red-green even when I show the code all at once.

## Adding a Feature

Do you remember that the initial feature - the building of the Hash of Arrays - was just the tip of a feature iceberg?
Then you learned that there were a lot of other features that needed to be considered.
Now you're dealing with real-world messiness — and this is where test-driven development shines.

I'm going to implement the rest of the features using tests to drive development, and I'll start by addressing the `nil` emails in the production data.
Along the way, I'm going to move the test data into a `setup` method and do a little refactoring:

```rb
# ./app.rb

require 'minitest'
require 'minitest/autorun'
require 'minitest/reporters'

Minitest::Reporters.use!

class UserEmailGrouperTest < Minitest::Test
  def setup
    # make a basic sample and expectation available to all tests
    @users = [
      { name: "Jane", status: "active", email: "jane@example.com" },
      { name: "Tom",  status: "inactive", email: "tom@example.com" },
      { name: "Sara", status: "active", email: "sara@example.com" }
    ]
    @users_grouped_by_status = {
      "active" => ["jane@example.com", "sara@example.com"],
      "inactive" => ["tom@example.com"]
    }
  end

  def test_result_correctly_groups_emails_by_status
    assert_equal @users_grouped_by_status, UserEmailGrouper.new(@users).group_by_status
  end

  def test_result_excludes_nil_emails
    # Adding a user with nil email to @users
    @users << { name: "Bob", status: "active", email: nil }
    assert_equal @users_grouped_by_status, UserEmailGrouper.new(@users).group_by_status
  end
end

class UserEmailGrouper
  def initialize(users)
    @users = users
  end

  def group_by_status
    result = {}
    @users.each do |user|
      next if user[:email].nil? # skip if the email is nil
      result[user[:status]] ||= []
      result[user[:status]] << user[:email]
    end
    result
  end
end
```

Again, while presuming that we did this in "Red/Green" style, here's what changed:

- the test data is now in a `setup` block and exposed as instance variables
- method names updated to better reflect behavior
- implementation code updated to skip users with `nil` emails

And I have peace of mind that I can move forward because my tests pass.

Along the way, I got feedback from my tests several times.
Here's an example:

```sh
ERROR UserEmailGrouperTest#test_result_excludes_nil_emails
NameError: undefined local variable or method 'users' for an instance of UserEmailGrouperTest
            app.rb:27:in 'test_result_excludes_nil_emails'

ERROR UserEmailGrouperTest#test_result_correctly_groups_emails_by_status
NameError: undefined local variable or method 'users_grouped_by_status' for an instance of UserEmailGrouperTest
            app.rb:22:in 'test_result_correctly_groups_emails_by_status'
```

which reminded me that I needed to use instance variables, or this:

```sh
FAIL UserEmailGrouperTest#test_returns_hash_with_email_arrays_as_values_excluding_nil_emails
        --- expected
        +++ actual
        @@ -1 +1 @@
        -{"active"=>["jane@example.com", "sara@example.com"], "inactive"=>["tom@example.com"]}
        +{"active"=>["jane@example.com", "sara@example.com", nil], "inactive"=>["tom@example.com"]}
```

which gave me clear insight into how my code was transforming the User data.
Remember, test failures aren't a problem — they're the proof that the system is protecting itself and giving great feedback.
Welcome these messages!

**Why This Matters**

You might think that the example above is super trivial, and you're right!
And yet... how often have you found yourself wanting to refactor something or move code around in a simple script... and you can't.
Or you **do it and things blow up**, and you ask yourself "why?"
Then you start to litter your code with diagnostic outputs to try and understand where things are breaking down.
This has been my life more often than I care to think about.

Here's the question I didn't have a great answer for: **why _not_ lead with tests**?

This little refactor ended up with a tiny mental cost.
Low stress.
I moved from green to red, back to green again - confidently and quickly.
No drama, just momentum.

## Another Feature

I'm going to add another feature - the "opted_out" flag.

Remember, the `opted_out` flag is a little weird.
Sometimes it's there, sometimes it isn't, so my first instinct is to reach for Ruby's "safe navigation" operator (`&.`).

But... I'm also wondering whether I can use safe navigation **with** an equality check like `==`.
With tests I can make educated guesses, relying on Ruby's "principle of least surprise" design, and confirm whether my ideas check out:

```rb
# ./app.rb

require 'minitest'
require 'minitest/autorun'
require 'minitest/reporters'

Minitest::Reporters.use!

class UserEmailGrouperTest < Minitest::Test
  def setup
    @users = [
      { name: "Jane", status: "active", email: "jane@example.com" },
      { name: "Tom",  status: "inactive", email: "tom@example.com" },
      { name: "Sara", status: "active", email: "sara@example.com" }
    ]
    @users_grouped_by_status = {
      "active" => ["jane@example.com", "sara@example.com"],
      "inactive" => ["tom@example.com"]
    }
  end

  def test_result_correctly_groups_emails_by_status
    assert_equal @users_grouped_by_status, UserEmailGrouper.new(@users).group_by_status
  end

  def test_result_excludes_nil_emails
    # Adding a user with nil email to @users
    @users << { name: "Bob", status: "active", email: nil }
    assert_equal @users_grouped_by_status, UserEmailGrouper.new(@users).group_by_status
  end

  def test_result_excludes_emails_with_opted_out_field
    # Adding a user with opted-out status to @users
    @users << { name: "Alice", status: "active", email: "alice@example.com", opted_out: true }
    # The user we just added shouldn't be in the @users_grouped_by_status
    assert_equal @users_grouped_by_status, UserEmailGrouper.new(@users).group_by_status
  end
end

class UserEmailGrouper
  def initialize(users)
    @users = users
  end

  def group_by_status
    result = {}
    @users.each do |user|
      next if user[:email].nil?
      next if user[:opted_out]&.== true # Exclude opted-out users, and a funky method call
      result[user[:status]] ||= []
      result[user[:status]] << user[:email]
    end
    result
  end
end
```

Doing this red-green, we get a working test suite.

**Plot Twist - The Code Above Is Too Clever**.
As I worked through this code, I asked myself _do I even need safe navigation here?_

With the backstop of a working test suite, I have the safety and confidence to ask a simple question like that and get an instant answer.
I changed this line:

```rb
result[user[:opted_out]]&.== true
```

to:

```rb
result[user[:opted_out]] == true
```

And... the tests still pass.
Awesome.

It turns out Ruby hashes handle missing keys and `nil == true` just fine - returning false - which means I don't have to guard against `nil` explicity in the `Hash`, meaning safe navigation wasn't necessary.
The test suite confirmed it, and I got to delete an operator I didn't need.

This is what testing unlocks: **not just correctness, but curiosity**.
Without the backstop of a working test suite... would I have even asked that question?
Maybe, but also maybe not since the consequence of pursuing that sort of answer could introduce regressions and require _A Lot Of Work_.

## The Freedom to Refactor

I could keep going like this and implement all of the new features in the single method, but I'm starting to get uneasy about what `.group_by_status` is turning into.
A couple things stand out:

- That `result = {}` accumulator is a pretty sure sign that something else like an `.each_with_object({})` or a `.select()` can be used to clean things up.
- The filtering logic (aka, the `next`s) is doing too much inline work.  It's getting a little harder to read and track what's going on.  I'd like to turn those into methods.

At the moment, email validation is only checking whether email is `nil?`.
But I know how the world works: that logic will grow, and so will the logic for opting out.
Extracting those statements into their own methods feels like a good way to keep things sane and, more importantly, easy to read.

In a REPL or test-free coding context, a refactor is not a fun exercise, enough so that I just might not do it.
In the test-driven context I have confidence in my current code because I have tests that exercise it.
Therefore, all I need to do is make sure my refactored code passes the tests.

And... voilà!

```rb
# ./app.rb

require 'minitest'
require 'minitest/autorun'
require 'minitest/reporters'

Minitest::Reporters.use!

class UserEmailGrouperTest < Minitest::Test
  def setup
    # make a basic sample and expectation available to all tests
    @users = [
      { name: "Jane", status: "active", email: "jane@example.com" },
      { name: "Tom",  status: "inactive", email: "tom@example.com" },
      { name: "Sara", status: "active", email: "sara@example.com" }
    ]
    @users_grouped_by_status = {
      "active" => ["jane@example.com", "sara@example.com"],
      "inactive" => ["tom@example.com"]
    }
  end

  def test_result_correctly_groups_emails_by_status
    assert_equal @users_grouped_by_status, UserEmailGrouper.new(@users).group_by_status
  end

  def test_result_excludes_nil_emails
    # Adding a user with nil email to @users
    @users << { name: "Bob", status: "active", email: nil }
    assert_equal @users_grouped_by_status, UserEmailGrouper.new(@users).group_by_status
  end

  def test_result_excludes_emails_with_opted_out_field
    # Adding a user with opted-out status to @users
    @users << { name: "Alice", status: "active", email: "alice@example.com", opted_out: true }
    assert_equal @users_grouped_by_status, UserEmailGrouper.new(@users).group_by_status
  end
end

class UserEmailGrouper
  def initialize(users)
    @users = users
  end

  def group_by_status
    @users.each_with_object({}) do |user, result|
      next if user_email_is_invalid?(user)
      next if user_is_opted_out?(user)
      result[user[:status]] ||= []
      result[user[:status]] << user[:email]
    end
  end

  private

  def user_email_is_invalid?(user)
    user[:email].nil?
  end

  def user_is_opted_out?(user)
    user[:opted_out] == true
  end
end
```

The structure of my `UserEmailGrouper` changed dramatically - but the tests didn't.
And that's exactly the point.

I'm feeling better about this code: I've swapped in `.each_with_object({})`, which elimates the manual accumulator step.
I've extracted business logic into clearly named methods.
The code reads better.
It's easier to extend.
This is a kindness that I can gift to anyone who enters this codebase later.

**Tests are more valuable than you realize**.

A good test suite offers value to you right now, in the present moment, while you're developing.
Tests also bring confidence and support you while you refactor.
And critically, the same tests that support you in the present also support you in the future.

Here's a guarantee - you're going to forget about some implementation choices you made 8 months ago.
But your tests won't, and that makes them priceless.

## Feature Wrap Up

Time to finish up the last two requirements:

- Emails in each group should be sorted alphabetically.
- Duplicate users (by email address) should be excluded entirely.

As before, I can red-green through two new test cases, and then make my changes in the implementation:

```rb
# ./app.rb

require 'minitest'
require 'minitest/autorun'
require 'minitest/reporters'

Minitest::Reporters.use!

class UserEmailGrouperTest < Minitest::Test
  def setup
    # make a basic sample and expectation available to all tests
    @users = [
      { name: "Jane", status: "active", email: "jane@example.com" },
      { name: "Tom",  status: "inactive", email: "tom@example.com" },
      { name: "Sara", status: "active", email: "sara@example.com" }
    ]
    @users_grouped_by_status = {
      "active" => ["jane@example.com", "sara@example.com"],
      "inactive" => ["tom@example.com"]
    }
  end

  def test_result_correctly_groups_emails_by_status
    assert_equal @users_grouped_by_status, UserEmailGrouper.new(@users).group_by_status
  end

  def test_result_excludes_nil_emails
    # Adding a user with nil email to @users
    @users << { name: "Bob", status: "active", email: nil }
    assert_equal @users_grouped_by_status, UserEmailGrouper.new(@users).group_by_status
  end

  def test_result_excludes_emails_with_opted_out_field
    # Adding a user with opted-out status to @users
    @users << { name: "Alice", status: "active", email: "alice@example.com", opted_out: true }
    assert_equal @users_grouped_by_status, UserEmailGrouper.new(@users).group_by_status
  end

  def test_result_emails_are_sorted
    # add a user to the end of the list, should sort to the beginning
    @users << { name: "Alice", status: "active", email: "alice@example.com" }
    # add a user to the end of the list, should stay at the end
    @users << { name: "Zeek", status: "active", email: "zeek@example.com" }
    @users_grouped_by_status["active"].unshift("alice@example.com")
    @users_grouped_by_status["active"].push("zeek@example.com")
    assert_equal @users_grouped_by_status, UserEmailGrouper.new(@users).group_by_status
  end

  def test_result_excludes_users_with_duplicate_emails
    # Adding a user with a duplicate email to @users
    @users << { name: "Clone", status: "active", email: "jane@example.com" }
    # The duplicate user's email should be excluded from the result
    @users_grouped_by_status["active"].delete("jane@example.com")
    assert_equal @users_grouped_by_status, UserEmailGrouper.new(@users).group_by_status
  end
end

class UserEmailGrouper
  def initialize(users)
    @users = users
    @email_counts = Hash.new(0)
    @users.each { |user| @email_counts[user[:email]] += 1 if user[:email] }
  end

  def group_by_status
    @users.each_with_object({}) do |user, result|
      next if user_email_is_invalid?(user)
      next if user_is_opted_out?(user)
      result[user[:status]] ||= []
      result[user[:status]] << user[:email]
    end.transform_values(&:sort)
  end

  private

  def user_email_is_invalid?(user)
    user[:email].nil? || @email_counts[user[:email]] > 1
  end

  def user_is_opted_out?(user)
    user[:opted_out] == true
  end
end
```

I'm done, and I feel great, very low stress levels.

Maybe someone who comes after me raises an eyebrow at the `.transform_values()` tailing off the end of the `.each_with_object({})` in `group_by_status`.
But it does the job, and it keeps sorting logic out of the main loop.

Maybe more requirements emerge and I have to think about where to add more features.
Those are future concerns and in the present, I am happy and confident with the code I've written.
The logic is easy to verify and I can prove it works by exercising the test suite.
Even better: I'm ready to share my code with others without dread.
I'm not guessing, I'm not hoping.
I know what the code does and I can prove it.

## Bonus: Taking The Experience Up a Notch

Developer experience is a huge consideration in the Ruby community and testing is no exception.
There's great tooling around testing, and one of the best additions you can make to your testing setup is Guard.
It's not required for what I've done so far, but it takes the whole experience up a level.

Adding it to this project is as simple as:

```sh
% bundle add guard guard-minitest
```

Then create a Guardfile in your project directory, right at the same level as the `app.rb` file:

```rb
# frozen_string_literal: true

clearing :on

guard :minitest, all_after_pass: true, test_folders: ["."], test_file_patterns: "*.rb" do
  # with Minitest::Unit
  watch(/^app\.rb$/) { "./app.rb" }
end
```

and then run everything from the command line while editing in your favorite editor:

```sh
% bundle exec guard
```

This runs the tests every time I press save, allowing me to bounce back and forth in the same file, never losing my focus.
A small thing, but it makes a big difference in my workflows.

## Conclusion

Embracing testing transformed my relationship with my career, elevating my skills and reducing my stress.
I shipped better code faster when I fully embraced testing.
And I have every confidence it will do the same for you.

Even though you've seen how quickly we can set up a framework to run tests, you still might be tempted not to bother - especially if an idea seems _too simple_ to test.
To help you fight the temptation to REPL first, I wrote [Spiker](https://github.com/norlinga/spiker).
It's not hard to set up tests, as you've seen, but Spiker makes it **too easy** to do anything else.
Spiker handles every bit of boilerplate we worked through above - including Guard - and supports:

- basic, single-file style tests,
- multi-file testing for more complex ideas,
- a Minitest + "Given" flavored approach,
- a lightweight Rspec setup,
- and optional Dockerized coding environments

If you're curious, give it a spin.
Open a terminal and install the Spiker gem, type one command, and start exploring your next idea — under test.

> To My Past Self,
> 
> Testing not magic. It’s not even hard. It’s just the next step forward.
>
> Start small. Write one test. Make it pass. Then another. You’re not just writing tests. You’re building confidence. Momentum. Clarity.
>
> You can do this. You’re closer than you think. Try it. You're going to love it.
