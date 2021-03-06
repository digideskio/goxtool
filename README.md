#### Hosted version available at [https://zerogox.com](https://zerogox.com)

# Goxtool

Goxtool is a Python trading client and auto-trading bot for the MtGox Bitcoin currency exchange. It is designed to work in the Linux console and has a curses user interface. It can display live streaming market data and you can buy and sell with keyboard commands.

Goxtool also has a simple interface to plug in your own automated trading strategies, your own code can be reloaded at runtime, will receive events from the API and can act upon them.


## Installation

Open a terminal in an empty folder or in the folder you usually use to clone repositories and clone the master branch:

```
git clone git://github.com/caktux/goxtool.git
```

This will create a folder named goxtool containing all the needed files. Thats all, now it is installed and ready to use. You can now already watch live market data (without any trading functions being enabled), you can later add a MtGox API-key to it to have full access to your account but for now just proceed to the next step, start it without an account, just to make sure everything works.


## Usage

Change into to the goxtool folder that was created in the previous step and start goxtool.py:

```
cd goxtool
 ./goxtool.py
```

Keyboard commands (only the ones useful in view-only mode, without Mt.Gox account):

- <kbd>q</kbd> quit
- <kbd>l</kbd> (lower case "L") reload the strategy module (see advanced usage)
- <kbd>D</kbd> (shift + d) switch to depth chart view
- <kbd>H</kbd> (shift + h) switch to candlestick history chart view
- <kbd>S</kbd> (shift + s) toggle summing up the volume of order book levels  on/off
- <kbd>T</kbd> (shift + t) toggle summing up the volume in the depth chart on/off
- <kbd>-</kbd> order book zoom out (increase group size)
- <kbd>+</kbd> order book zoom in (decrease group size)
- <kbd>,</kbd> depth chart zoom out (increase group size)
- <kbd>.</kbd> depth chart zoom in (decrease group size)

(There will be even more commands once you connect it to your Mt.Gox account)

There is also a goxtool.ini file, it will be created on the first start. In the .ini file there are some parameters you can change, for example the currency you want to trade BTC against or some parameters regarding the network protocol. Some of the .ini settings can be overridden by command line options (use the --help option to see a list). The default protocol is now PubNub, with websocket and socketio as alternatives. Goxtool implements all three protocols with PubNub currently being more reliable. There is also an option to use the http API for trading commands with websocket and socketio, the default is to send all commands to the streaming socket when using websocket, this affects only what happens when you buy/sell/cancel, it does not affect the streaming update of order book and chart.

The two options are socketio or websocket, the .ini setting for this is use_plain_old_websocket. To force it connecting with socketio:

```
./goxtool.py --protocol=socketio
```

 To force it connecting to the websocket server do this:

```
./goxtool.py --protocol=websocket
```

It has turned out that websocket is currently the most reliable protocol after pubnub. These options on the command line take precedence over what you have configured in the .ini file (but it won't change your .ini), you can make websocket the default (so you don't need this option anymore) by editing the ini file.

If you experience a high lag between sending an order and the ack (the op:result) not appearing immediately which is happening during times of really high volume then you should consider also adding --use-http to make it send trading commands via http. Http makes it slightly slower to execute many trades in a row but has been much more reliable when mtgox was under heavy load or ddos or similar. If you don't need to send many orders very fast then this option won't hurt. --use-http can be combined with either socketio or websocket and is forced for PubNub.

The following is what I am currently using and I recommend it:

```
./goxtool.py --protocol=websocket --use-http
```


## Trading with your MtGox account

First you will need to add an API key in MtGox, then do the following:

```
./goxtool.py --add-secret
```

This will now ask you for your key, secret and a password (not your MtGox one) to secure those on your drive. The key and secret belong to a shared secret that is created by MtGox to authenticate your trading software against their API. You can request as many keys from MtGox as you need, every application you connect to your MtGox account should have its own key, you can also at any time delete the keys again that you no longer need.

If you need a Key/Secret pair for goxtool, open your web browser, log in to your MtGox account, click on "Security Center", click on "Advanced API Key Creation", choose a name for your key (every key will have a name so you can later easily tell them apart), make sure you check at least the boxes "Get Info" and "Trade" (this is what the application will be allowed to do with this key) and then finally click on "Create Key".

Now MtGox will create 2 strings of cryptic numbers and letters, the "API-Key" and the "Secret". Now copy/paste the API-Key into the terminal where it asked you for "Key", press enter, now it will ask you for "Secret", copy/paste the Secret into the terminal, press enter and now it will ask you for a passphrase. It is important to understand what is going on here. The Key/Secret from above must under no circumstances ever come into the wrong hands, therefore goxtool won't just store them in the .ini file, goxtool will encrypt it and thats what the passphrase is needed for. Choose a secure passphrase (it will ask you twice to make sure there is no typo), you will also not see anything in the console (not even "*") while typing, this is not a bug, this is intentional. Choose a strong passphrase, type it into the terminal, press enter, repeat the passphrase, press enter again and now it will tell you that it has been encrypted and saved to the .ini file and exit.

Now start goxtool again:

```
./goxtool.py
```


Which will ask:

```
enter passphrase for secret:
```

From now on every time you start goxtool it will ask you for the passphrase in order to be able to decrypt and use the secret. Enter your passphrase, press enter. Now goxtool will start and you will notice that now it is showing your account balance at the top of the window. Now all trading functions are enabled.

Keyboard commands for trading:

- <kbd>F4</kbd> : New buy order
- <kbd>F5</kbd> : New sell order
- <kbd>F6</kbd> : View orders / cancel order(s)

In the cancel dialog you can move up/down with the arrow keys, use INS or = to select/unselect orders (you can select multiple orders and cancel them all at once) or if you just quickly want to cancel only one order just highlight to the order and hit F8. It behaves a little bit like deleting files in midnight commander.

When entering a new order you can move between the fields with up/down keys or move to the next field with tab or enter (but only if you entered a valid number into the previous field, decimal separator is . (not comma, even on European computers), send the order with enter.

All dialogs can be closed with `F10` or `ESC`.


## Strategy modules

Running all strategies:

```
./goxtool.py --strategy=balancer.py,buy.py,sell.py
```


#### Balancer

Portfolio rebalancing bot that will buy and sell to maintain a constant asset allocation ratio of exactly 50/50 = fiat/BTC.

- <kbd>i</kbd> for information (how much currently out of balance)
- <kbd>o</kbd> to see order book
- <kbd>r</kbd> to rebalance with market order at current price (required before rebalancing)
- <kbd>p</kbd> to add initial rebalancing orders
- <kbd>c</kbd> to cancel all rebalancing orders
- <kbd>u</kbd> to update account information, order list and wallet
 
```
./goxtool.py --strategy=balancer.py
```

#### Buy strategy

Buy strategy module. Set `buy_level` at the price you want to buy, `threshold` above your level for a log alert and `volume` in fiat (`0` for full balance). Set `simulate` to `False` to activate.

* <kbd>b</kbd> to see Buy objective

```
./goxtool.py --strategy=buy.py
```

#### Sell strategy

Sell strategy module. Set `sell_level` at the price you want to sell, `threshold` below your level for a log alert and `volume` in fiat (`0` for full balance). Set `simulate` to `False` to activate.

- <kbd>s</kbd> to see Sell objective

```
./goxtool.py --strategy=sell.py
```

#### Making your own

You can write your own trading bots. There is a file named `strategy.py`, it contains a class Strategy() which constitutes a trading bot that by default does nothing (its an empty skeleton). It has event methods (slots) connected to signals that will be fired when certain events occur. From within these methods you can then do arbitrary stuff (peek around in gox.orderbook to see where bids and asks are located, call gox.buy(), gox.sell()  or gox.cancel() methods to build a fully automated trading bot or you can use the key press slot (it will be called on all letter keys except l and q) to build a semi-automatic bot that reacts to key presses or to influence parameters of your bot or anything else you can imagine. Examples of simple bots will soon follow.

If you decide to make serious use of this then please create a new python file for your strategy. either make a copy of the default strategy.py skeleton or make a module that imports strategy and has a class Strategy(strategy.Strategy), give this module file a different name and leave strategy.py alone so it won't collide with upstream changes you pull from github. By default goxtool will load strategy.py but you can start it with the --strategy command line option to specify your own strategy module or a comma separated list of many modules:

```
./goxtool --strategy=mybot.py,myotherbot.py
```

You can even edit the strategy while goxtool is running and then reload it at runtime (this can be very useful), just press the l key (lowercase L) and it will do the following things:

* emit signal _strategy_unload, this will call slot_before_unload()
* free the currently running instance of Strategy() (your __del__() method should be called)
* re-import the changed module file
* create a new instance of Strategy() and call your__init__() again.

You should persist the state of your bot (if needed) in slot_before_unload() and reload it in __init__(). Leave the __del__() method alone, its only there to print a log message to debug proper unloading!

Please make sure that you can see the debug output from the __del__() method in the log when the strategy is reloading, you must be sure its able to free and garbage collect your strategy! If you instantiate any circular references, even something as innocent as a double linked list or even just an object holding a reference to the strategy then this will effectively keep python from being able to garbage-collect it and hold it in memory indefinitely (and keep sending it signals!).

Use the slot_before_unload() method to del everything in your strategy that might hold any circular references. You can check that it works if you see the debug output  of __del__() in the log scrolling by when you press l to reload it, the fact that __del__() was called is proof that it was properly garbage-collected.

Trading functions do NOT block, this means they also won't return the MtGox order ID, you need to find your own way of remembering which orders you have sent already. A few moments (seconds or minutes) after you have sent them they will be acked by MtGox and it will fire orderbook.signal_changed()and when this happens you will find it in the gox.orderbook.owns list and it will have an official order ID. I know this is not optimal (because this part of the code is not yet complete, eventually there will be dedicated signals to notify your bot about the results of trading commands) and also this document is not yet a complete documentation. If you really want to dive into this: use the source, Luke.
How to keep it up to date

Occasionally I will commit bugfixes, improvements, etc. To update your copy of goxtool (assuming you previously installed it with git clone and not by just downloading a zip file) do the following:

```
git pull
```

and if that complains because of local uncommitted changes because you edited the strategy.py module or did other changes to the code then try this:

```
git stash
git pull
git stash pop
```

Of course you could have also have followed my previous advise to not do anything other than simple throw-away experiments in strategy.py so you can always `git reset --hard` if everything else fails and use a separate file (and probably even separate git branches) for serious bot development but this is outside the scope of this document, there exist specialized howtos for git and github elsewhere.

Original user manual is here:
[http://prof7bit.github.com/goxtool/](http://prof7bit.github.com/goxtool/)

##### Donations appreciated

prof7bit (goxtool): 1C8aDabADaYvTKvCAG1htqYcEgpAhkeYoW

caktux (mods): 18zX3wb318o2Pw9ZUHgG3mmQME536Qg2Ha
