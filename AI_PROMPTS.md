# Generative AI Disclosure

Using AI tools for this challenge is allowed. Be honest here — an accurate
"I used AI heavily" is worth more to us than an inaccurate "I did not".

**Did you use AI tools?**: yes

**Which ones?**:
ChatGPT, Codex (in VSCode), Claude

---

## 1. Key prompts
List the main prompts you used. Paraphrasing is fine.
1. Be a devops tutor. Help me debug the assignment, the main goal is that I need to know how to explain every step I take. Help me debug using VSCode, pointing out the important parts of the error message. Ask me diagnostic questions to quiz my understanding. 
2. Explain this Docker build error and identify the lines that point at the root of the error. 
3. "Explain which parts of this configuration are syntax, commands, or names, and what each does."
4. What are a few ways to check if the fix worked? Do I need to request something to see the error?

---

## 2. Where the AI got it wrong

Describe **one concrete case** where the AI's answer was wrong, outdated, or
sub-optimal. Cover: what it suggested, how you noticed it was wrong, and what
you did instead. 3–5 sentences.

During troubleshooting, I ran `bash ./scripts/run-tests.sh` from PowerShell and
first saw `line 5: $'\\r': command not found`, caused by Windows CRLF line
endings in the shell script. After changing the file to LF, the `bash` command
selected WSL and reported `The command 'docker' could not be found in this WSL
2 distro.` ChatGPT kept steering me toward WSL integration, but WSL was not
needed for this project: Docker Compose worked from PowerShell, and GitHub
Actions runs the script on Ubuntu. I used PowerShell for local Docker testing
and treated the WSL message as a consequence of the command I chose, not as a
project defect.
---

## 3. Anything in your submission you could not fully explain?

We would rather know. Write "nothing" if that is the case.

I understand the four main fixes. I have not fully worked through all the configuration changes required to run Nginx as a non-root user.
