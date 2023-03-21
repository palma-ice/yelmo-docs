# How to use Jupyter Notebook over ssh

### Step 1

Open a terminal, enter the remote machine and run:

```
ssh user@snowball.fis.ucm.es
jupyter notebook --no-browser --port 1235
```

Here port `1235` is chosen, but another port could be used too.
In the remote terminal, this message should appear:

```
http://localhost:1235/?token=LARGERANDOMNUMBER
```

### Step 2

Open another terminal on the local machine and run:

```
ssh -L 1235:localhost:1235 user@snowball.fis.ucm.es
```

IMPORTANT: use the same port as chosen in step (1).

### Step 3

Go back to the remote terminal and copy the link shown into a browser
on the local machine. The Jupyter Notebook running on the remote machine should now
be open in a local browser. Enjoy!
