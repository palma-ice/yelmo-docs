# How to use Jupyter Notebook over ssh

### Step 0

Make sure you have Jupyter installed on the remote machine.
To do so, login to the remote machine via ssh and run:

```bash
pip3 install jupyterlab
```

If it succeeds without errors, then it should be working.

### Step 1

On the remote machine, open a Jupyter Notebook instance by running:

```bash
jupyter notebook --no-browser --port 1235
```

Here port `1235` is chosen, but another port could be used too.
In the remote terminal, this message should appear:

```bash
http://localhost:1235/?token=LARGERANDOMNUMBER
```

### Step 2

Open another terminal on the local machine and run:

```bash
ssh -L 1235:localhost:1235 user@snowball.fis.ucm.es
```

IMPORTANT: use the same port as chosen in step (1).

### Step 3

Go back to the remote terminal and copy the link shown into a browser
on the local machine. The Jupyter Notebook running on the remote machine should now
be open in a local browser. Enjoy!
