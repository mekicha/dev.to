You have heard of the [subprocess][subprocess] module in the Python Standard Library, haven't you? How it:
> allows you spawn new processes, connect to their input/output/error pipes, and obtain their return codes.

Not only that, it also intends to replace older modules and functions like `os.system` and `os.spawn*`, which to be honest, I have no experience using.

Basically, the subprocess module allows you to launch other programs from your code and drive them to whatever end you desire. Think of doing fun things like:

1. controlling a browser from your code via the [webbrowser][webbrowser] module or even a third party library like [selenium](https://github.com/SeleniumHQ/selenium)
2. automating server deployments or administration tasks with [Fabric](http://www.fabfile.org/)
3. Running an android emulator from your code.
4. Yeah, I mean, you get the idea!

The [documentation][subprocess] has all the information you need if you desire a deep dive. If that is you, thank you for coming :)

Those of us who just want to stand by the riverbank and scoop some water are the ones reading this sentence and the ones below it.

Jokes aside, what brought us to this point is that I have a couple of repositories I collaborate on with other colleagues. All these repos live in the `Projects` subdirectory inside my home directory. The goal is to be able to run a `git pull` on all those repos at the beginning of my work day automatically. The other alternative will be to `cd` into a project directory, run `git pull`, `cd` out and into the next one, etc. That, my friends, is very boring!

Without much further ado, here is the code:
 
```python

from subprocess import check_output, CalledProcessError
from pathlib import Path

project_dir = Path.home() / 'Projects' / 'work'

for d in project_dir.iterdir():
    if d.is_dir(): # this check is not really needed
        try:
            out = check_output(["git", "pull", d], cwd=d)
            res = out.decode()
            print(res.splitlines()[-1])
        except CalledProcessError as exc:
            pass

```

In a nutshell, I am going through all the directories(repos) in my work directory, running a `git pull` on them. I am also decoding the output(by default output is in bytes) and printing the last element in the output list(calling res.splitlines turns the output into a list).
If there is an error, I pass it like it didn't happen.

That is all, believe me! I can run that little script from anywhere and have my repos updated. 
Maybe I can even take this further by checking the output to see if there was really an update on the repo. If so, I can open that particular repo in a code editor or on a browser. But what's the point?



[subprocess]: https://docs.python.org/3/library/subprocess.html
[webbrowser]: https://docs.python.org/3.6/library/webbrowser.html#