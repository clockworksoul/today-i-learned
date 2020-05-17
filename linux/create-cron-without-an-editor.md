# How to Create a Cron Job Using Bash Without the Interactive Editor

_Source: [Stack Overflow](https://stackoverflow.com/questions/878600/how-to-create-a-cron-job-using-bash-automatically-without-the-interactive-editor)_

To create a cron job without using the interactive editor, you do one of the following:

### Generate and install a new crontab file

1. Write out the current crontab to a file

We use `crontab -l`, which lists the current crontab jobs.

```bash
crontab -l > mycron
```

2. Append the new cron to the file

```bash
echo "00 09 * * 1-5 echo hello" >> mycron
```

3. Install the new cron file

```bash
crontab mycron
```

4. Clean up

```bash
rm crontab
```

### As a one-liner

Alternatively, you can do this in a one liner by piping your modified cron file to `crontab -` as follows:

```bash
crontab -l | { cat; echo "0 0 0 0 0 some entry"; } | crontab -
```

Broken down:

* `crontab -l` lists the current crontab jobs
* In a subshell, `cat` prints the `crontab -l` output and echos the new command
* The entire output is piped to `crontab -` which installs it as a new crontab.
