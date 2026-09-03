# yupyter to pdf

commands i ran after having jupyter installed:

``` bash
sudo apt install pandoc texlive-xetex texlive-fonts-recommended texlive-plain-generic

jupyter nbconvert --to <output format> <input notebook>
jupyter nbconvert --to pdf obligatory/O1/workspace/O1.ipynb
```

it created a new file in same folder as target file

using command nbconvert inside jupyter



