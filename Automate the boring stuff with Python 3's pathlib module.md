The `Pathlib` module was introduced in Python 3.4 with the aim:
> to provide a simple hierarchy of classes to handle filesystem paths and the common operations users do over them.

In this article, I intend to go through some of these common operations. Towards the end, I show how I keep the downloads folder on my computer clean and organized using the `Pathlib` module and some cron job. Let us get started.

As per the [documentation][documentation]:
> If you’ve never used this module before or just aren’t sure which class is right for your task, Path is most likely what you need. It instantiates a concrete path for the platform the code is running on.

That is understood. Let's import the `Path` class then.  
 <pre><font color="#008700">In [</font><font color="#8AE234"><b>9</b></font><font color="#008700">]: </font><font color="#008700"><b>from</b></font> <font color="#0087D7"><b>pathlib</b></font> <font color="#008700"><b>import</b></font> Path
</pre>

If you are following along with an interactive shell like [Ipython](https://ipython.org/) or any other, it is usually a good idea to `dir` the imported class or object to get a clue on the attributes and methods the class provides.

### Get the user home directory
```python
# returns PosixPath('/home/mekicha')
# This is equivalent to os.path.expanduser('~')
home_dir = Path.home()

``` 
 
### Get a directory in the home directory
```python
# returns PosixPath('/home/mekicha/Downloads')
# Equivalent to: home_dir.joinpath('Downloads')
downloads_dir = home_dir / 'Downloads'

downloads_dir.exists() # True

```

Now, at this point, you might be thinking: "Is this all there is to this click-baiting article that promised to show me practical examples of using the Pathlib module?". Before you close your tab in fury at the waste of time and bandwith, I offer a calm answer to your urgent question: "No, friend, that is not all".

In fact, if that was all, I would have just pointed you to the [documentation][documentation] where such and more examples abound.
I just provided the examples above as some form of a ground work for the better things ahead(hopefully). 
Now, what do you think about the following examples.

### Given a directory, calculate the total size of all files

```python
from pathlib import Path

def get_size(directory: Path):
  """ Given a directory, return total size of files in bytes """
  total_size = 0
  for file in directory.iterdir():
     total_size += file.stat().st_size  # in bytes
  return total_size

if __name__ == '__main__':
  # Let's get the size of my home directory
  directory = Path.home()
  size_in_byte = get_size(directory)
  
  # Nice to have: convert the size to megabyte or gigabyte

  mega = 1 << 20
  giga = 1 << 30 
  if size_in_byte / giga < 1:
    if size_in_byte / mega < 1:
      print(f'file size = {size_in_byte} bytes')
    print(f'file size = {size_in_byte / mega} megabytes')
  print(f'file size = {fize_in_byte / giga} gigabytes'

```


### Given a directory, return the file with the biggest size
```python
def get_max_file(directory: Path):
  return max((f.stat().st_size, f) for f in directory.iterdir())

# In the same vein, let's get last modified file in a directory

def get_last_modified(directory: Path):
  return max((f.stat().st_mtime, f) for f in directory.iterdir())

```

### Given a directory, return a list of subdirectories in the directory
```python
def get_dir_list(directory: Path):
  return [child for child in directory.iterdir() if child.is_dir()]

```

### Organizing my computer's download folder

If you have read this far, hopefully, this is the beginning of what you came for.

My download folder used to be a mess. Files of different formats a lie side by side with one another in a helpless disorderly manner, each one crying for my attention, and perhaps feeling neglected when I did not click on them. Not anymore. Today, help has come their way.  
I am going to put the files in their places, where they should belong. Images should go to the images sub-folder, documents(think pdf, doc, xls) should live in the documents folder and it is right for zipped files to dwell in the zipped folder. I will assign a cron agent to do the job.


The cron agent would be setup to run once every day, at a time when I should be asleep.

Here is the code:

```python
#!usr/bin/env python3

import time
import logging
from pathlib import Path

logging.basicConfig(level=logging.DEBUG)

logger = logging.getLogger(__file__)


def organize_directory(directory: Path):
    """ Take a path to a directory and organizes the files there
        into subdirs.
     """

    sub_dirs = ['images', 'docs', 'zipped','media', 'others']
    images = ['.png', '.jpeg', '.jpg', '.gif']
    zipped = ['.zip', '.gz', 'tgz']
    docs = ['.doc', '.docx', '.pdf', '.xls', '.xlxs', '.PPT', '.txt']
    media = ['.mp4', '.mkv', '.avi', '.mp3']

    # create base subdirs.
    # Map each subdir to its subdir path
    dir_map = {}
    for subdir in sub_dirs:
        subdir_path = directory.joinpath(subdir)
        subdir_path.mkdir(parents=True, exist_ok=True)
        dir_map[subdir] = subdir_path

    start_time = time.monotonic()
    for child in directory.iterdir():
        child_name = child.name
        if child.is_file():
            if child.suffix in images:
                target = dir_map['images'].joinpath(child_name)
                child.replace(target)
                logger.info(f'moved {child} to images folder')
            elif child.suffix in docs:
                target = dir_map['docs'].joinpath(child_name)
                child.replace(target)
                logger.info(f'moved {child} to docs folder')
            elif child.suffix in zipped:
                target = dir_map['zipped'].joinpath(child_name)
                child.replace(target)
                logger.info(f'moved {child} to zipped folder')
            elif child.suffix in media:
                target = dir_map['media'].joinpath(child_name)
                child.replace(target)
                logger.info(f'moved {child} to media folder')
            else:
                target = dir_map['others'].joinpath(child_name)
                child.replace(target)
                logger.info(f'moved {child} to others folder')

    end_time = time.monotonic() - start_time
    logger.info(f'Organizing directory took {end_time} seconds')
    


if __name__ == '__main__':
    download_dir = Path.home().joinpath('Downloads')

    organize_directory(download_dir)


```

Again, the documentation entries for [iterdir][iterdir], [mkdir][mkdir] and [replace][replace] are very clear on what they do.

Running the above script gave me this:

`INFO:download_folder_agent.py:Organizing directory took 0.01179114996921271 seconds`

That's not all. It has also given me a much better organized downloads folder.

### Setup the cron job

Setting up the cron job was fairly easy.

Here are the steps I took that gave me what I wanted:
1. Open the crontab editor by typing: `crontab -e `
2. Look up the crontab schedule expression on [crontab-guru][crontab guru]. I choose everyday at 5 am. 0 5 * * *
3. Adding the following job entry in the file:
`0  5 * * * python3 /home/mekicha/agent/download_folder_agent.py >> /home/mekicha/agent/log 2>&1
4. I saved the changes and exited the file.
5.Voila! Everything was working.

Actually, I first set the schedule to run every minute * * * * *, checked that it was working before I committed the above schedule.

I set the schedule to run every day by 5 am, which is not really ideal. It's not like I am downloading stuff every now and then. But I will leave it at that for now. Feel free to choose whatever seems best to you.


If you are wondering what does this 2>&1 thing mean, this [stackoverflow](https://stackoverflow.com/questions/818255/in-the-shell-what-does-21-mean) answer was what I read when I was wondering that too.


Okay, I think this is a good place to bring this whole business to an end.
In the next episode of automating the boring stuff, I will show how I keep the downloads folder clean by removing files that have outlived their usefulness.

Thank you!

P.S. This automate the boring stuff article series is an imitation of [a book by the same name](https://automatetheboringstuff.com/).


[documentation]: https://docs.python.org/3.5/library/pathlib.html
[iterdir]: https://docs.python.org/3.5/library/pathlib.html#pathlib.Path.iterdir
[mkdir]: https://docs.python.org/3.5/library/pathlib.html#pathlib.Path.mkdir
[replace]: https://docs.python.org/3.5/library/pathlib.html#pathlib.Path.replace
[crontab-guru]: https://crontab.guru/examples.html