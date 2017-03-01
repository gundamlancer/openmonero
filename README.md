# Open Monero - a fully open sourced implementation of MyMonero

In this example [restbed](https://github.com/Corvusoft/restbed/) is used to
 provide Monero related JSON REST service. To demonstrate this, 
 a service called Open Monero was developed.


Open Monero is an open source implementation of backend of
https://mymonero.com/. The frontend, which includes HTML, CSS, JavaScript, was adapted
from (and originally developed by) https://mymonero.com/. 

Unlike MyMonero, Open Monero's backend is open sourced, free
to use, host and modify. Additionally, some features were added/changed. 
They include:

 - google analytics, cloudflare, images and flash were removed.
 - transaction fees were set to zero (MyMonero also has now them zero due to problem with its RingCT).
 - the wallets generated use 25 word mnemonics, fully compatible with official monero wallets
(13 word mnemonics generated by MyMonero work as usual though).
 - import wallet fee was reduced.
 - added support of testnet network and mainnet network without relying transactions
 - improved handling of mempool, coinbase, locked and unlocked transactions.
 - added dynamic fees for testnet.
 - minimum mixin set to 4 for the next hard fork.
 - view only mode added.
 - ability to offline show your address and private view and spend keys, based on the mnemonic seed,
  in case of server or connections problem.
   
## Testnet version

- [http://139.162.32.245:81](http://139.162.32.245:81)

This is Open Monero running on testnet network. You can use it to play around with it. 
Since this is testnet version, frequent changes and database resets are expected. Also,
 it is running on cheapest vps, which may result in some lag.

If you want some testnet monero, please make issue with your testnet address that you can
obtained from Open Monero.   
   
## Screenshot

![Open Monero](https://raw.githubusercontent.com/moneroexamples/openmonero/master/screenshot/screen1.png)


## Host it yourself

The Open Monero consists of four components that need to be setup for it to work:

 - MySql/Mariadb database - it stores user address (viewkey is not stored!), 
 associated transactions, outputs, inputs and transaction import payments information.
 - Frontend - it is virtually same as that of MyMonero, except before mentioned differences.
  It consists of HTML, CSS, and JavaScript.
 - Monero daemon - daemon must be running and fully sync, as this is 
 where all transaction data is fetched from and used. Daemon also commits txs 
 from the Open Monero into the Monero network.
 - Backend - fully written in C++. It uses [restbed](https://github.com/Corvusoft/restbed/) to serve JSON REST to the frontend 
 and [mysql++](http://www.tangentsoft.net/mysql++/) to interface the database. It also accesses Monero blockchain and "talks"
 with Monero deamon.
    
   
## Limitations

#### Performance

Open Monero is not as fast as MyMonero. This is because it is basic, easy to understand
 and straight forward implementation of the backend. Thus, it does not use any catching
 of transactions, blocks, complex database structures and SQL queries. Also, no ongoing 
 monitoring of user's transactions is happening, since viewkey is not stored. Transaction
 search threads start when user logs in (viewkey and address are submitted to the search thread), 
 and finish shorty after logout. Once the search threads stop, 
 they can't be restarted without user logging in back, because viewkey is unknown.



## Example setup on Ubuntu 16.04 

Below are example and basic instructions on how to setup up and run Open Monero on Ubuntu 16.04. 


#### Monero libraries

Monero's libraries and header files are setup is described here:

- [compile-monero-on-ubuntu-16-04](https://github.com/moneroexamples/compile-monero-09-on-ubuntu-16-04)

#### Compilation of the Open Monero (don't run it yet)

Download Open Monero and compile it. In fact we could postpone compilation to later, but 
we can just do it now, to see if it compiles. But don't run it yet. It will not
work without database, setup frontend, and synced and running monero blockchain.

```bash
# need mysql++ libraries 
sudo apt install libmysql++-dev 

git clone https://github.com/moneroexamples/openmonero.git

cd openmonero

mkdir build && cd build

cmake ..

make
```

#### Mysql/Mariadb

```bash
sudo apt install mysql-server
sudo mysql_secure_installation
```

Download `openmonero.sql` provided and setup the `openmonero` database. `openmonero.sql` script will
drop current `openmonero` if exist. So don't run it, if you have already some important information
in the `openmonero` database.

Assuming we are still in `build` folder:

```bash
# apply it to mysql
mysql -p -u root < ../sql/openmonero.sql
```

#### Lighttpd and frontend

```bash
sudo apt-get install lighttpd
```
Assuming you are still in `build` folder, copy frontend source files into lighttpd www folder.

```bash
sudo mkdir /var/www/html/openmonero
sudo cp -rvf ../html/* /var/www/html/openmonero/
```

Setup document root in `lighttpd.conf` into openmonero folder

```bash
sudo vim /etc/lighttpd/lighttpd.conf
```

and change `server.document-root` into:

```bash
server.document-root    = "/var/www/html/openmonero"
```

Restart lighttpd to see the change:

```bash
sudo systemctl restart lighttpd
```

Go to localhost (http://127.0.0.1) and check if frontend is working.


#### Run Open Monero

Command line options

```bash
./openmonero -h

  -h [ --help ] [=arg(=1)] (=0)         produce help message
  -t [ --testnet ] [=arg(=1)] (=0)      use testnet blockchain
  --do-not-relay [=arg(=1)] (=0)        does not relay txs to other nodes. 
                                        useful when testing construction and 
                                        submiting txs
  -p [ --port ] arg (=1984)             default port for restbed service of 
                                        Open Monero
  -c [ --config-file ] arg (=./config/config.json)
                                        Config file path.
```

Before running `openmonero`: 

 - make sure you have `Downloads` folder in your home directory. 
 Time library used in Open Monero stores there time zone offsets database that it uses.
 - edit `config/confing.js` file with your settings. Especially set `frontend-url` and `database`
 connection details.
 - set `apiUrl` in `html\js\config.js` and `testnet` flag. Last slash `/` in `apiUrl` is important. 
 If running backend for testnet network, frontend `testnet` flag must be set to `true`.
 For mainnet, it is set to `false`.
 - make sure monero daemon is running and fully sync. If using testnet network, use daemon
 with testnet flag!
   

To start for mainnet: 
```bash
./openmonero
```

To start for testnet: 
```bash
./openmonero -t
```
   
To start for testnet with non-default location of `config.json` file:

```bash
./openmonero -t -c /path/to/config.js
```

## Scrap notes (just for myself)

### Generate your own ssl certificate 
 
Setting up https and ssl certificates in restbed
 - https://github.com/Corvusoft/restbed/blob/34187502642144ab9f749ab40f5cdbd8cb17a54a/example/https_service/source/example.cpp

Based on the link above:
 
```bash
# Create Certificate
cd /tmp
openssl genrsa -out server.key 1024
openssl req -new -key server.key -out server.csr
openssl x509 -req -days 3650 -in server.csr -signkey server.key -out server.crt
openssl dhparam -out dh2048.pem 2048
```
 
### API calls

Check if Open Monero REST service is working 

```bash
curl  -w "\n" -X POST http://139.162.32.245:1984/get_version
```
Example output:
```
{"last_git_commit_date":"2017-02-25","last_git_commit_hash":"f2008aa","monero_version_full":"0.10.2.1-release"}
```

## Other examples

Other examples can be found on  [github](https://github.com/moneroexamples?tab=repositories).
Please know that some of the examples/repositories are not
finished and may not work as intended.

## How can you help?

Constructive criticism, code and website edits are always good. They can be made through github.
