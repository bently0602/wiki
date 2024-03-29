# Install Ruby

[https://phoenixnap.com/kb/install-ruby-ubuntu
](https://github.com/postmodern/chruby)

https://github.com/postmodern/ruby-install#readme

## Install chruby

```
cd ~
mkdir ~/.chruby
wget https://github.com/postmodern/chruby/releases/download/v0.3.9/chruby-0.3.9.tar.gz
tar -xzvf chruby-0.3.9.tar.gz -C ~/.chruby --strip-components=1
cd ~/.chruby
make install
```

## Install install-ruby

```
cd ~
mkdir ~/.ruby-install
wget https://github.com/postmodern/ruby-install/releases/download/v0.9.1/ruby-install-0.9.1.tar.gz
tar -xzvf ruby-install-0.9.1.tar.gz -C ~/.ruby-install --strip-components=1
cd ~/.ruby-install
make install
```

## Add to home (Bash only)

Ubuntu:

```
echo 'source /usr/local/share/chruby/chruby.sh' >> ~/.bashrc
echo 'export PATH="$HOME/.ruby-install/bin:$PATH"' >> ~/.bashrc
```

Other Bash:

```
echo 'source /usr/local/share/chruby/chruby.sh' >> ~/.bash_profile
echo 'export PATH="$HOME/.ruby-install/bin:$PATH"' >> ~/.bash_profile
```

Then source it so current shell is sees new HOME:

```
source ~/.bashrc
```

## Install ruby

See avail to install:

```
ruby-install
```

Install (by default to /opt/rubies):

```
ruby-install ruby 3.2.2
```

See avail to switch:

```
chruby
```

Switch to:
```
chruby 3.2.2
```
