[![hackmd-github-sync-badge](https://hackmd.io/YIWaeRG_SriNtTCLaNXx-w/badge)](https://hackmd.io/@hayat01sh1da/creation-of-a-shorthand-command-to-generate-pull-request-title-with-label-from-a-topic-branch-name)

<img src="https://hackmd.io/_uploads/S1U-uKSdC.jpg" alt="Ruby" />

## 1. Background

I have been tired of manually inputing a Pull Request Title like `[school-career-research] Fix Bug Causing Arguments Error when Attendance Number Is Empty` based on the corresponding topic branch whose name is `hayat01sh1da/issue-74921/school-career-research/fix_bug_causing_arguments_error_when_attendance_num_is_empty`.  
I thought about how efficient it is if I can get a wanted Pull Request title once I run a shorthand command taking its topic branch name as the argument.  
Actually, I succeeded in lowering my stress level as to it and making things regarding Pull Requests tidier.  
Implementation and its design are so simple that anyone who knows Ruby and Linux commands can do it incredibly easily, so I would like to share how in the hope that it makes someone else happy.

## 2. Requirements

The following 3 elements need extracting for the Pull Request title.

- `hotfix`(optional): We create this kind of topic branch if we must deploy a patch implementation to the production environment. `hotfix` is its marker.
- `school-career-research`: Microservice name to make differences in. As to the Pull Request title, it should be surrounded by`[]` to be treated as a label.
- `fix_bug_causing_arguments_error_when_attendance_num_is_empty`: Branch topic. It should represent what is realised in the topic branch and articulate why the differences are needed. It ought to be capitalised and treated as a Pull Request topic.

Pull Request title is created by the label and topic generated above.

## 3. Ruby Script Implementation

1. Create a Ruby file(e.g. `~/branch_name_to_pr_title.rb`).
2. Receive a string from the command-line argument, extract the target branch name and sustain it in an instance variable.
3. Split the branch name by `/` delimiter and make a string array.
4. Decide how many elements to extract from the string array checking if `hotfix` is included as the 3rd element in it.
    - if included: Extract `hotfix`, the micro-service name and the branch topic
    - unless included: Extract the micro-service name and the branch topic
5. If `hotfix` is included, convert it to `Hotfix` by capitalisation and treat it as the label combined with the micro-service name.
6. Split the branch topic by the delimiters, either `-` or `_` and make a string array, then capitalise each string element. Finally, combine them into a string splitted by white spaces and treated as the Pull Request topic.
7. Combine the label generated at the phrase5 and the Pull Request topic at the phrase6.
8. Output the full Pull Request title by `Kernel.#p` in the console you are watching.

```ruby
class BranchNameToPrTitle
  def self.print_pr_title(branch_name)
    self.new(branch_name).branch_name_to_pr_title
  end

  def initialize(branch_name)
    @branch_name = branch_name
  end

  def branch_name_to_pr_title
    string_array            = branch_name.split(/\//)
    *prefixes, branch_topic = string_array[2] == 'hotfix' ? string_array[-3..] : string_array[-2..]
    "#{labels(prefixes)}#{pr_topic(branch_topic)}"
  end

  private

  attr_reader :branch_name

  def labels(prefixes)
    if prefixes.include?('hotfix')
      prefixes.first.capitalize!
      "[#{prefixes.join('][')}] "
    else
      "[#{prefixes.last}] "
    end
  end

  def pr_topic(branch_topic)
    branch_topic.split(/[\-\_]/).map(&:capitalize).join(' ')
  end
end

branch_name, *_ = ARGV

p BranchNameToPrTitle.print_pr_title(branch_name)
```

## 4. Register an alias in either ~/.bash_profile or ~/.zprofile

Simplify `branch_name_to_pr_title` to `b2p` and register it as the alias as follows.  
`git branch --show-current` prints the current branch name and it is passed to `xargs` via the pipe `|`, and then the Ruby script finally receive it as the command-line argument.

```command
# WSL and Linux
echo 'alias b2p="git branch --show-current | xargs ruby ~/branch_name_to_pr_title.rb"' >> ~/.bash_profile
source ~/.bash_profile

# MacOS
echo 'alias b2p="git branch --show-current | xargs ruby ~/branch_name_to_pr_title.rb"' >> ~/.zprofile
source ~/.zprofile
```

## 5. Execution

Pass the topic branch name to `b2p` as the command-line argument and it prints the wanted Pull Request title.  
The rest of what you have got to do is just to copy and paste it in the Pull Request title section.

```command
# For the normal cases
% git branch --show-current
hayat01sh1da/issue-74921/school-career-research/fix_bug_causing_arguments_error_when_attendance_num_is_empty
% b2p
"[school-career-research] Fix Bug Causing Arguments Error When Attendance Num Is Empty"
% git branch --show-current
hayat01sh1da/issue-74921/school-career-research/fix-bug-causing-arguments-error-when-attendance-num-is-empty
% b2p 
"[school-career-research] Fix Bug Causing Arguments Error When Attendance Num Is Empty"
% git branch --show-current
hayat01sh1da/issue-74921/school-career-research/fix_bug-causing_arguments-error_when-attendance_num-is_empty
% b2p 
"[school-career-research] Fix Bug Causing Arguments Error When Attendance Num Is Empty"

# For the Hotfix cases
% git branch --show-current
hayat01sh1da/issue-74921/hotfix/school-career-research/fix_bug_causing_arguments_error_when_attendance_num_is_empty
% b2p 
"[Hotfix][school-career-research] Fix Bug Causing Arguments Error When Attendance Num Is Empty"
% git branch --show-current
b2p hayat01sh1da/issue-74921/hotfix/school-career-research/fix-bug-causing-arguments-error-when-attendance-num-is-empty
% b2p 
"[Hotfix][school-career-research] Fix Bug Causing Arguments Error When Attendance Num Is Empty"
% git branch --show-current
hayat01sh1da/issue-74921/hotfix/school-career-research/fix_bug-causing_arguments-error_when-attendance_num-is_empty
% b2p 
"[Hotfix][school-career-research] Fix Bug Causing Arguments Error When Attendance Num Is Empty"
```

## 6. Conclusion

I am sorry, but it does not keep function words such as prepositions and conjunctions.  
If I tried to realise it, I would take bigger pains which do not pay.  
I am not a big fan of it, so I am sure that this simple implementation is the best so far.  
Another fault is the implementation supposes `hosfix` comes in the 3rd element delimited by `/`, so the Ruby script will not handle it at all once the naming style is out of the rule.
