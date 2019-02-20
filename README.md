slidenumbers: true

# [fit] Automate all the things

---

# [fit] あらゆる作業を自動化しよう

---

# Self Intro

- Joshua Kaplan
- minne @ GMO Pepabo
- Danger-Swift contributer
- Likes 🤖

---

# [fit] CI is the ultimate automation tool

---

# [fit] Why?

---

- Fewer mistakes
- Faster
- More efficient
- More exciting

---

# Key CI tools

- Fastlane
- Danger
- SwiftLint

---

# [fit] Normal CI flow (level 1)

---

# Run tests for each PR

---

![100%](images/test_results.png)

---

# [fit] Intermediate CI flow (level 2)

---

# [fit] Linters and other static analysis

---

# SwiftLint + Danger

```rb
swiftlint.lint_files inline_mode: true
```

---

![100%](images/danger_warning.png)

---

# Check storyboards

---

```rb
# storyboardにずれているビューや制約の不備がないか
parsed_diff.each do |changed_file|
  next unless changed_file.file.match(/.*\.(storyboard)$/)

  changed_file.changed_lines.each do |changed_line|
    content = changed_line.content
    has_misplaced_view = content.include? 'misplaced="YES"'
    has_ambiguous_view = content.include? 'ambiguous="YES"'

    file = changed_file.file
    line_number = changed_line.number

    if has_misplaced_view
      fail('ずれているビューがあるので、Update Framesを実行してください', file: file, line: line_number)
    end
    if has_ambiguous_view
      fail('制約が不十分な箇所があるので、直してください', file: file, line: line_number)
    end
  end
end
```

---

# Spell checking

---

![100%](images/spell_check.png)

---

```rb
# タイポを検知する
added_and_modified_files = git.added_files + git.modified_files
added_and_modified_files.each do |file_path|
  next unless file_path =~ /\.swift$/
  stdout, status = Open3.capture2("npx", "cspell", file_path)

  next if status.success?
  stdout.split("\n").each do |line|
    next unless matches = /\w+\.swift:(\d+).*-\sUnknown\sword\s\((\w+)\)/.match(line)
    line_number = matches[1].to_i
    word = matches[2]

    warning = "タイポかも？ #{word}"
    warn(warning, file: file_path, line: line_number)
  end
end
```

---

# [fit] Automate PR tasks

---

# Assign reviewers

---

```rb
# レビュワー指定（コメント指定のためにこれを一番上に書く必要あり）
reviewers = ["user1", "user2", "user3"].reject { |reviewer| reviewer == github.pr_author }
repo_name = github.pr_json["head"]["repo"]["full_name"]
pr_number = github.pr_json["number"]

number_of_comments = github.api.issue_comments(repo_name, pr_number).size
if number_of_comments.zero?

  reviewers = reviewers.sample(2)

  github.api.request_pull_request_review(
    repo_name,
    pr_number,
    {},
    "reviewers": reviewers
  )
end
```

---

# Post CI results

---

![100%](images/xcode_summary.png)

---

```rb
# Xcode Summary
build_report_file = 'build_results.json'
xcode_summary.ignored_files = 'Pods/**'
xcode_summary.ignores_warnings = true
xcode_summary.inline_mode = true
xcode_summary.report build_report_file
```

---

# Post code coverage

---

![100%](images/xcov_danger.png)

---

```rb
xcov.report(
  scheme: 'minne',
  workspace: 'minne.xcworkspace',
  exclude_targets: 'TodayExtension.appex, NotificationServiceExtension.appex, MinneKit.framework',
)
```

---

# [fit] Advanced CI flow (level 3)

---

# Custom SwiftLint rules

---

```yml
prefer_gregorian_calendar:
  name: "Gregorian Calendar"
  regex: "Calendar\\.current"
  message: "Please use `Calendar(identifier: .gregorian)` to avoid Japanese calendar-related bugs"
  severity: error
https_only:
  name: "HTTPS Only"
  match_kinds:
    - string # コメントなどを無視して、文字列のみをみる
  regex: "http:"
  message: "Please use HTTPS due to security policy"
  severity: error
```

---

# Deliver beta and release builds

---

```rb
lane :release do
  capture_screenshots                  # generate new screenshots for the App Store
  sync_code_signing(type: "appstore")  # see code signing guide for more information
  build_app(scheme: "MyApp")
  upload_to_app_store                  # upload your app to App Store Connect
  slack(message: "Successfully uploaded a new App Store build")
end
```

---

# Automate screenshot delivery

---

```swift
// App
let app = XCUIApplication()
setupSnapshot(app)
app.launch()
```

```rb
# Fastlane
lane :screenshots do
  capture_screenshots
  frame_screenshots(white: true)
  upload_to_app_store
end
```

---

# Check for dead code

---

## ![100%](images/dead_code.png)

---

```rb
def get_dead_objc_code
  std_out, status = Open3.capture2("bundle", "exec", "fui", "--ignorexib", "--path", "minne/Classes")
  # fuiのexitstatusは、使われていないクラスの数で、
  # Bridging-Headerが必ず当たってしまうので、
  # 2つ以上の結果がある場合通知している。
  return status.exitstatus >= 2 ? std_out : ""
end

def get_dead_swift_code
  std_out, _ = Open3.capture2("periphery", "scan")
  r = /minne.*(Struct|Class) .*is unused$/
  filtered_results_array = std_out.to_enum(:scan,r).map {$&}.flatten
  return filtered_results_array.join("\n")
end

# Main
dead_objc_result = get_dead_objc_code
dead_swift_result = get_dead_swift_code
mention = "<!subteam^S1DDSFQSF|minne-ios>"
message = "#{mention}, Unused files exist:\n"

notify_slack(message + dead_objc_result) unless dead_objc_result == ""
notify_slack(message + dead_swift_result) unless dead_swift_result == ""
```

---

# Release tag generation

---

![100%](images/release_tags.png)

---

![90%](images/release_notes.png)

---

# [fit] Speed optimization

---

# Rome

- https://github.com/blender/Rome

---

# Dynamic code coverage

---

# [fit] Conclusion

---

# [fit] Automate all the things
