# Murphy

Murphy is a script that keeps a shadow git repository for your working directory, but commits every 15 seconds (by default).

To start using, install it, and run `murphy` inside your repository. Murphy will record all your working directory changes until you stop it.


## Instalation

Murphy requires Perl, but has zero dependencies outside the core. It should just run out-of-the-box.

Download the file `murphy`, place it on some directory on your `PATH`, and make it an executable.

For example:

    curl -LO ...
    sudo bash -c 'cp murphy /usr/local/bin && chmod 555 murphy /usr/local/bin/murphy'

Note: on some Linux distributions, you might get an error like `Can't locate GetOpt/Long.pm in @INC (you may need to install the GetOpt::Long module)`. This is caused by the brain-dead decision of splitting Perl into two packages, the core and the library, against Perl policy. Use `apt-get install -y perl-core` or `yum install -y perl-core` to fix it.


## How to use

By default Murphy runs in the foreground, and prints one line of status with the date and the output of `git diff --shortstat` of the per commit.

You can silence the status lines with `--quiet`.

Murphy checks for new changes and commits any found every 15 seconds. You can change this with `--every`. For example, `murphy --every 5` will commit every 5 seconds, and `murphy --every 1m` will commit every minute.

Tip: you can use `d`, `h`, `m` and `s` as suffixes for you time value. Default is `s`.

You can also start Murphy in the background with `--daemon`. If you want to stop it later, use `--stop` inside the same repository.

If you don't like the idea of leaving Murphy processes running all over the place, you can use `--bored` and pass it time value. If no changes are made to your repository after that amount of time passes, Murphy will exit. For example, `murphy --bored 24h`, will make Murphy terminate if you go 24 hours without any changes. See Automation below for more options.

Tip: all long options can be abbreviated. `-q` is equivalent to `--quiet`, `-e` equivalent to `--every`, and so on.

Given that Murphy is just a git repository, you can run any git commands directly with it. You can use `murphy log` to look at the latest changes. The entire arguments are passed through to `git`, so `murphy log --oneline` works as expected.


## Automation

You don't want to forget to run Murphy, right?

I recommend you look at smartcd package. It provides per-directory hooks, scripts that run whenever you enter a specific directory (or any subdirectory of that directory).

On my "enter" scripts, I use the following command: `murphy --daemon --bored 48h`.


## How it works?

The first time it runs, Murphy creates a new bare git repository inside the current repository `.git` directory. Murphy uses the directory `org.simplicidade.murphy` inside the `.git` folder.

Inside that folder you'll find several files and directories:

* `repo`: the bare repository used by Murphy;
* `sock`: a UNIX pipe that Murphy listens to for commands, used to prevent multiple copies from running and to stop it in case of daemon mode;
* `last`: the output of `git show --stat` of the last commit.

Every N seconds, it runs `git add -A .` and `git commit` at the root of your repository, with the environment `GIT_DIR` pointing to the Murphy repository.

If Murphy is not running as a daemon nor in quiet mode, it will print the current timestamp (local time) plus the output of `git show --shortstat`.


## License

This software was written in August 2016 by [Pedro Melo](https://www.simplicidade.org), and is licensed under the [The Artistic License 2](https://opensource.org/licenses/Artistic-2.0).
