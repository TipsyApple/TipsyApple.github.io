# Some Tricks About Bash/Zsh/Fish...
Yestorday, I learned some tricks about **\*sh** when I try to run my program. The tricks I used are as follows:

## 1. ```wc``` which means **Word Count**:
```wc -l``` which gives the number of lines in the standard output:
```bash
some_command_that_can_be_output_by_line | wc -l 
```
For example:
- Input: ```ls -l | wc -l```
- Ouput: ```5``` the total number of the files in the current directory

P.S. When using upper case...em...I mean using ```wc -L``` which you can get the number of bytes in the longest row. It's true, I've tried.😂

## 2. 
```bash
tail -n 14 nohup.out | grep -A 7 "iter:"
```
Looks a bit complicated. Don't worry. I will explain one by one:

```
tail FILE
``` 
You can use ```man tail``` to see:
> print the last 10 lines of each FILE to standard output.

Ugh... it seems that I didn't explain anything except copying the manual. I remain indifferent to it. just for fun. Don't say something useless, continue.

```
tail -n NUM FILE
```
output the last ```NUM``` lines.

Of course, you can use it like this ```ls -l | tail -n 1``` which print the last line of this command ```ls -l``` output.

P.S. The usage of ```head``` is the same, except that the output is the first few lines of the file. for instance:
- Input: ```ls -l | tail -n 1```
- Output: ```drwxr-xr-x     7 tipsy   224  5  7 14:43 LabelVOC```

```
grep -A NUM PATTERN
```
```A``` means ```After Context```, so you can also use ```--after-context=NUM``` instead.
> Print NUM  lines  of  trailing  context  after  matching  lines.

```
grep -B NUM PATTERN
```
```B``` means ```Before Context```. It can instead of ```--before-context```. Yep, similar to above.
> Print NUM lines of leading context before matching lines.

```
grep -C NUM PATTERN
```
```C``` means ```Context```. I don't wanna say more. It just makes the above two commands work together.
> Print NUM lines of output context.

## 3. 
```bash
nohup unzip /dir/zipfile.zip -o -d /dir_you_want > unzip.info.txt 2>&1 &
```
hah, more complicated than last one. 

```
nohup
``` 
one of the shell buildin commands which maybe means ```no hungup```. it sets the signal SIGHUP to be ignored. As a result, it will not be terminated when you logout from ssh. In short:
> Run COMMAND, ignoring hangup signals.

> If standard output is a terminal, append output to 'nohup.out' if possible, '$HOME/nohup.out' otherwise. If standard error is a terminal, redirect it to standard output.

```
unzip /dir/zipfile.zip -o
```
can creates ZIP archives. ```-o``` means **overwrite** existing files without promping. ```-d /dir_you_want```: An optional directory to which to extract files. By default, will be created in the current directory. ```

```
> redirected_standard_output_file
```
It redirects standard output to this file, overwriting the file.

P.S. ```>>``` will not overwrite instead of appending the redirected output at the end.

```
2>&1
```
seems mystery, but not at all. ```2``` and ```1``` just are two of **[file descriptor](https://en.wikipedia.org/wiki/File_descriptor)**. A file descriptor is a non-negative integer. ```2``` represents standard error, while ```1``` represents standard output. the last number ```0``` represents standard input. the '&' tells system '1' represents standard output rather than file named '1'. 
> Writing just ```2>1``` would redirect the standard error to a file called "1", not to standard output.[[2]](https://unix.stackexchange.com/questions/89386/what-is-symbol-and-in-unix-linux)

P.S. ```2>&1``` call the ```dup2(1,2)```, I could not tell you more, search it if interest.

```
&
```
the last symbol which can start the program as a background job.

P.S. ```fg``` foreground. It can bring a background job back to the foreground. (but if you've redirected output you won't see much.)[[3]](https://serverfault.com/questions/41959/how-to-send-jobs-to-background-without-stopping-them)

### TO BE CONTINUE
These days, I was exhausted. wanan.

References:
1. [IBM developerWorks](https://www.ibm.com/developerworks/cn/linux/l-cn-nohup/index.html)

2. [sergut On unix.stackexchange.com](https://unix.stackexchange.com/questions/89386/what-is-symbol-and-in-unix-linux)

3. [cas On serverfault.com](https://serverfault.com/questions/41959/how-to-send-jobs-to-background-without-stopping-them)
