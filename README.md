# sagclick
```
export HUGO_INST_DIR=/mnt/c/Ake/Apps/hugo

cd $HUGO_INST_DIR
wget https://github.com/gohugoio/hugo/releases/download/v0.128.2/hugo_extended_0.128.2_Linux-64bit.tar.gz
mkdir 0.128.2.lnx
tar -xvf hugo_extended_0.128.2_Linux-64bit.tar.gz -C 0.128.2.lnx
ln -s 0.128.2.lnx latest

HUGO_HOME=$HUGO_INST_DIR/latest

# Possible 404 error
cd themes/
git submodule add https://github.com/nanxiaobei/hugo-paper

# check the files in local
$HUGO_HOME/hugo server -D

# build the pages
$HUGO_HOME/hugo
$HUGO_HOME/hugo -b http://sag.click
```