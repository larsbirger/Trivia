# Jupyter to pdf

1. install prereuisites

    ```Bash
    sudo apt install pandoc texlive-xetex texlive-fonts-recommended texlive-plain-generic
    ```

    ```Bash
    python3 -m pip install venv
    ```

2. create, activate and set up up environment

    1. create environment

        ```Bash
        python3 -m venv .venv
        ```

    2. activate environment

        ```Bash
        source .venv/bin/activate
        ```

    3. setting up environment
        1. jupyter

            ```Bash
            pip install jupyter
            ```

        2. nbconvert

            ```Bash
            pip install nbconvert
            ```

3. run command to convert jupyters `.ipynb` to `<output format>`
    
    ```Bash
    jupyter nbconvert --to <output format> <input notebook>
    ```

    like here, where it converts the file `O1.ipynb` to `.pdf`

    ```Bash
    jupyter nbconvert --to <output format> <input notebook>
    jupyter nbconvert --to pdf obligatory/O1/workspace/O1.ipynb
    ```

    it creates a new file in same folder as target file


