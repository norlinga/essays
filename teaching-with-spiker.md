# Teaching and Assessing Ruby Developers — on Their Own Terms

I'm the author of [Spiker](https://github.com/norlinga/spiker), a Ruby project scaffolding gem for test-driven exploration. I wrote Spiker in the conviction that the best way to *learn* is to *build*. Spiker helps Ruby developers get *building* fast, with just enough structure to support clean iteration, testing, and discovery.

Recently, I had a job interview where the format was to build code in response to a series of tests added by the interviewer. The idea was solid — test-driven interviewing — but the process was clunky. Tests were added manually. Screen sharing and control-passing slowed everything down. I couldn't use the tools or editor I was most comfortable with.

That experience got me thinking: why shouldn't an interview or learning session be more like a real-world dev workflow?

Enter Spiker — again.

---

## The Problem

Whether you’re trying to *assess* someone’s coding ability — or just *teach* them a new skill — you'd ideally provide a system that is:

- Easy to set up
- Aligned with your actual workflows
- Non-intrusive to the user's environment
- Progressive and testable

Common online assessment and teaching platforms force learners into browser sandboxes or require local installation of unfamiliar tooling and all the headaches that can bring. Browser sandboxes can be amazing, and yet *don’t* let the student use their favorite editor, terminal setup, keybindings, or test runner.

---

## The Spiker-Based Solution

With a little bit of structure and a Rake task, Spiker can be turned into a lightweight learning engine.

Here’s how it works:

### 1. Scaffold a Project with Spiker

Generate a project scaffold:

```bash
$ spiker multi teaching
```

...that's it.

---

### 2. Add the `next` Rake Task to Your Project

Add the following to the Rake task to automatically run tests and "unlock" the next challenge if the current one passes:

```ruby
# Rakefile
require "fileutils"

desc "Run tests and load next challenge if all pass"
task :next do
  sh "bundle exec ruby test/test_helper.rb" # or `rspec` for RSpec users

  if $?.success?
    puts "✅ All specs passed!"

    next_spec = Dir["next/*.rb"].sort.first
    if next_spec
      FileUtils.mv(next_spec, "test/")
      puts "🧪 Next challenge loaded: #{File.basename(next_spec)}"
    else
      puts "🎉 No more specs left. You did it!"
    end
  else
    puts "❌ Specs failed. Try again!"
  end
end
```

This assumes you're using Minitest in `test/`, but you can swap it out with `bundle exec rspec` and rename folders to `spec/` as needed.

---

### 3. Organize Tests into Steps

Place test files that should be introduced in sequence into a `next/` directory:

```
next/
  001_addition_test.rb
  002_subtraction_test.rb
  003_division_edge_cases_test.rb
```

Initially, these are *disabled* simply by virtue of being outside the test runner's path. The student might be running tests continuously, using Guard, but as the test expectations they can run the rake task to test and proceed:

```
$ bundle exec rake next
```

As the student completes each test, the Rake task moves the next challenge into the live test folder.

---

### 4. Share the Repo

Now, just commit the project and share it:

```bash
$ git init
$ git add .
$ git commit -m "Initial challenge"
```

Push to the remote and share with students or interviewees. Whatever tools make them productive — VSCode, Vim, tmux, Docker, whatever.

---

## Why This Works

This pattern gives **structure without friction**:

* ✅ Students work in their own local environment
* ✅ No need for shared- or browser-based editors
* ✅ Progress is test-driven and self-paced
* ✅ The entire experience models real-world TDD

---

## Conclusion

If you're interested in trying this pattern, you can [get started with Spiker here](https://github.com/norlinga/spiker). Here's a sample [project that implements this pattern](https://github.com/norlinga/teaching-with-spiker) based on the [imaginary use case](https://github.com/norlinga/essays) in the "Testing Journey" essay.

---

**Happy hacking.**