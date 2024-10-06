# DevOps Fundamentals

In the second CodeCamp’24 session, I delved into the fundamentals of DevOps, focusing on the essentials of Git, shell scripting, and virtualization. I learned about the core functionalities of Git, such as distributed development, branching, merging, and data integrity, which are crucial for version control and collaboration among developers. The session also covered Git commands for basic snapshotting, managing branches, and handling remote repositories, along with more advanced techniques like cherry-picking and rebasing. Additionally, I explored the basics of Linux and shell scripting, which provided me with valuable skills for navigating directories, manipulating files, and automating tasks through scripts. The introduction to virtualization explained how it enables efficient use of hardware resources by creating virtual environments, a key concept in modern IT infrastructure. Overall, this week provided a solid foundation in essential DevOps practices, tools, and concepts. 
In the case part, I handled two cases about script writing and fundamental git operations. 
The first case was to create a script that searches for a specific word in a file and counts its occurrences. Read the word and the file as an input. Additional Step: Check if the file exists. 


# Case 1

> Create a script that searches for a specific word in a file and counts its occurrences. Read the word and the file as an input. Additional Step: Check if the file exists.


## Answer

To achieve this, we should first create a file named ``` script ``` (or another name ) and type the contents with the following commands:

```
touch script
vim script

```
Inside the script, we must first add the following line to access bash:

```
#!/bin/bash

```
Then we can give user messages and get inputs with the following lines:

```
read -p "Enter the file path: " file_path
read -p "Enter the word to search: " word
```
Alternatively, we can first use ``` echo ```, then use ``` read ```

After that, we need to check whether the file exists to search word in an appropriate file with the following lines:

```
if [[ ! -f $file_path ]]; then
    echo "File '$file_path' does not exist."
    exit 1
fi

```
Then, we use ``` grep ``` to count word count in a file like below:

```
word_count=$(grep -o -i "\\b$word\\b" "$file_path" | wc -l)
echo "The word '$word' occurs $word_count times in the file '$file_path'."

```
After writing the script, you will get the following error when you run with ``` ./script ```:

```
zsh: permission denied: ./script
```
To avoid this, we need to change permission to use script like below:

```
chmod +x script
```
You can run the script and find the word count. Enjoy!!!

## Full Script

```
#!/bin/bash

# Read the inputs
read -p "Enter the file path: " file_path
read -p "Enter the word to search: " word

# Check if the file exists
if [[ ! -f $file_path ]]; then
    echo "File '$file_path' does not exist."
    exit 1
fi

# Count the occurrences of the word 
word_count=$(grep -o -i "\\b$word\\b" "$file_path" | wc -l)
echo "The word '$word' occurs $word_count times in the file '$file_path'."

```

## Example Output:
```
Enter the file path: script
Enter the word to search: read
The word 'read' occurs        3 times in the file 'script'.

```

# Case 2

> Create a script that writes 100 lines of GOOD and BAD (select a random number to start writing BAD) and commits it 100 times. Bisect it to see where it starts to write BAD

## Answer

To achieve this, we should first create a file named ``` script2 ``` (or another name) and type the contents with the following commands:

```
touch script2
vim script2

```
Inside the script2, we must first add the following line to access bash:

```
#!/bin/bash

```
Then we need to init new repo like below:

```
mkdir new_repo
cd new_repo || exit
git init
```

After that, we need to create a file path to store ```GOOD``` and ```BAD``` words:

```
file_path="output.txt"
> "$file_path"

```
Then, we use ``` RANDOM ``` to assign a random number where the ```BAD``` is written:

```
start_bad=$((RANDOM % 100 + 1))

```
After that, we creates a for loop to write ```GOOD``` or ```BAD``` in each iteration and commit it like below:

```
for ((i=1; i<=100; i++)); do
    if (( i == start_bad )); then
        echo "BAD" >> "$file_path"
    else
        echo "GOOD" >> "$file_path"
    fi

    # Add and commit the changes
    git add "$file_path"
    git commit -m "Commit $i"
done

echo "Script finished. Good bye..."

```

After writing the script, you will get the following error when you run with ``` ./script2 ```:

```
zsh: permission denied: ./script2
```
To avoid this, we need to change permission to use script like below:

```
chmod +x script2
```
You can run the script and find the first commit where BAD is written with ```git bisect```. The detailed ```bisect``` usage is shown at the last section. Enjoy!!

## Full Script

```
#!/bin/bash

# Create a new git repository
mkdir new_repo
cd new_repo || exit
git init

# Create the file
file_path="output.txt"
> "$file_path"

# Assign random number where the "BAD" will be written
start_bad=$((RANDOM % 100 + 1))

# Write 100 lines and commit in each iteration
for ((i=1; i<=100; i++)); do
    if (( i == start_bad )); then
        echo "BAD" >> "$file_path"
    else
        echo "GOOD" >> "$file_path"
    fi

    # Add and commit the changes
    git add "$file_path"
    git commit -m "Commit $i"
done

echo "Script finished. Good bye..."

```

## Example Output:
```
Initialized empty Git repository in /Users/alphantulukcu/Desktop/OBSS/bad_good_repo/.git/
[main (root-commit) df2bd28] Commit 1
 1 file changed, 1 insertion(+)
 create mode 100644 output.txt
[main d6d8309] Commit 2
 1 file changed, 1 insertion(+)
[main 2f0c3d3] Commit 3
 ... 
[main 4b348d5] Commit 97
 1 file changed, 1 insertion(+)
[main 815d43e] Commit 98
 1 file changed, 1 insertion(+)
[main a7836f9] Commit 99
 1 file changed, 1 insertion(+)
[main 0fe7764] Commit 100
 1 file changed, 1 insertion(+)
Script finished. Good bye...

```
## Bisect Usage:
> Note: In this example, the index of BAD is 98!


After running the ```./script2```, you can start the bisect operations with the following line:

```
% cd new repo
% git bisect start
status: waiting for both good and bad commits
```
Then, you determine a commit reference (you can find a reference with ```git log --oneline```), and use it in bisect:

```
% git bisect bad
status: waiting for good commit(s), bad commit known
% git bisect good 9fdf072
Bisecting: 15 revisions left to test after this (roughly 4 steps)
[a11f393ff5f044f1e5c28249b3f78abb15f044eb] Commit 84
% git bisect good
Bisecting: 7 revisions left to test after this (roughly 3 steps)
[b005bd8bd5f01955fde9e8985e433d1d7b3b4956] Commit 92
% git bisect good
 Bisecting: 3 revisions left to test after this (roughly 2 steps)
[7e5c042b086e0571354ac52cac47f48aa14f6a94] Commit 96
% git bisect good
Bisecting: 1 revision left to test after this (roughly 1 step)
[815d43e77bcc68e1ebd60de495590a29f731aebc] Commit 98
% git bisect bad 
Bisecting: 0 revisions left to test after this (roughly 0 steps)
[4b348d51f25bfa463ac17541dedf777145d901a0] Commit 97
% git bisect good
815d43e77bcc68e1ebd60de495590a29f731aebc is the first bad commit
commit 815d43e77bcc68e1ebd60de495590a29f731aebc
Author: alphantulukcu <alphantulukcu@gmail.com>
Date:   Tue Jul 23 16:14:34 2024 +0300

   Commit 98

output.txt | 1 +
1 file changed, 1 insertion(+)
```
As you see, we can find which commit contains the first ```BAD``` with ```bisect``` command. To reset ```bisect```, you can type ```git bisect reset```.

