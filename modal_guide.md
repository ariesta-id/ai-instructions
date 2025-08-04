Modal volume is remote.

```py
import modal
import pandas as pd
import numpy as np

volume_name = "volume-name"
mount_path = "/mnt/" + volume_name
modal_volume = modal.Volume.from_name(volume_name, create_if_missing=True)
```

Define `app`, it will be used as functions decorator.

```py
app = modal.App(image=modal.Image.debian_slim(python_version="3.12")
```

What we import in the script above, we have to put it (except the `modal` library itself) in the app using chained methods like so:

```py
app = modal.App(
    image=modal.Image.debian_slim(python_version="3.12")
    .uv_pip_install("pandas", "numpy")
)
```

We can also add library only for the remote server without installing it locally by not putting the imports globally (prefer this to minimize installations locally):

```py
# no need for global imports
# import pandas as pd
# import numpy as np

with app.imports():
    import pandas as pd
    import numpy as np
```


And if we want to import another module (py file) in local, we can add it on remote by:

```py
app = modal.App(
    image=modal.Image.debian_slim(python_version="3.12")
    .uv_pip_install("pandas", "numpy")
    .add_local_python_source("local_module") # local_module.py
```

This is the typical remote function definition using Modal:

```py
@app.function(
    volumes={
        mount_path: modal_volume, # can have multiple volumes
    },
    timeout=1 * 60 * 60 # 1 hour
)
def remote_function(x, y, z):
    answer = (x * y) + z
    # Write answer in modal volume as output.txt
    with open(f"{mount_path}/output.txt", "w") as f:
        f.write(str(answer))
    modal_volume.commit() # after done writing files
```

And ONLY IF the files or computations are possibly large, we can reserve resources like this:

```py
@app.function(
    volumes={
        mount_path: modal_volume,
    },
    timeout=1 * 60 * 60,
    memory=1024 * 8, # 8 GB RAM 
    cpu=2, # 2 CPUs
    gpu="A10G" 
)
```

The following is the local entry point. 

```py
@app.local_entrypoint()
def main():
    remote_function.remote(1,2,3)

if __name__ == "__main__":
    with modal.enable_output():
        with app.run(detach=False): # detach=True to keep running even if disconnect 
            main()
```

Besides `.remote` there is also `.map` and `.starmap`. `.map` is used if a function only accepts 1 parameter. Modal's `.map()` is different from Python's `map()`.

```py
@app.function()
def my_func(a, b):
    return a + b

@app.local_entrypoint()
def main():
    assert list(my_func.starmap([(1, 2), (3, 4)])) == [3, 7]
```