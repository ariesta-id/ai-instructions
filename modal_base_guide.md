# Modal Boilerplate

If you are being asked to set a Modal Boilerplate, it means to incorporate 3 things:
- modal_base.py
- the modal-enabled python script top lines
- a function in the modal-enabled script
- the modal-enabled script main block

## Modal Base Python script `modal_base.py`

Check if there is modal_base.py in project directory. If not, create it, like so:

```py
import modal


def get_image(pip_installs: list[str], apt_installs: list[str] = []):
    # base_pip_install = ["pyyaml"]
    # pip_installs.extend(base_pip_install)
    modal_image = modal.Image.debian_slim()
    if apt_installs:
        modal_image = modal_image.apt_install(*apt_installs)
    return (
        modal_image.uv_pip_install(*pip_installs)
        .add_local_python_source("modal_base")
        # .add_local_dir("py_modules_dir", remote_path="/")
    )
```

## Modal boilerplate top lines

In the script of interest, we would put:

```py
import modal
from modal_base import get_image

modal_image = get_image(
    [
        "numpy",
        "pandas",
        # and other pip installs needed
    ]
)

with modal_image.imports():
    # Imports in this block for remote server only
    # Remote-server only imports are highly preferred
    import pandas as pd
    import numpy as np

app = modal.App(name="app_name", image=modal_image)

vol_name = "vol_name"
vol_path = f"/mnt/{vol_name}"
modal_vol = modal.Volume.from_name(vol_name, create_if_missing=False)

volumes_dict = {
    vol_path: modal_vol,
}
```

## Modal function

If the user said a function, e.g. `the_function()` is a Modal remote function, then set it up like so:

```py
@app.function(volumes=volumes_dict, timeout=24 * 60 * 60)
def the_function():
    # This will be executed remotely
    pass
```

## Main boilerplate

```py
@app.local_entrypoint()
def main():
    the_function.remote()


if __name__ == "__main__":
    with modal.enable_output():
        with app.run(detach=False):
            main()
```