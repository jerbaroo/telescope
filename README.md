<div align="center">

# Telescope
[![Build Status](https://img.shields.io/github/workflow/status/jerbaroo/telescope/Test)](https://github.com/jerbaroo/telescope/actions?query=workflow%3ATest)
[![Netlify Status](https://api.netlify.com/api/v1/badges/b42ff31b-1036-424b-8f24-419de5b62549/deploy-status)](https://app.netlify.com/sites/telescope-hs/deploys)
[![Documentation](https://img.shields.io/badge/-documentation-5e5086)](https://telescope-hs.netlify.app)
[![GitHub Stars](https://img.shields.io/github/stars/jerbaroo/telescope?style=social)](https://github.com/jerbaroo/telescope)

</div>

*Status: Working prototype / demo. But no longer in development.*

Focus moved to a non-Reflex specific variant at
[database-generic](https://github.com/jerbaroo/database-generic).

## Introduction

![Example GIF](https://github.com/jerbaroo/telescope/blob/master/example.gif?raw=true)

A Haskell library for Reflex apps that react to changes to data in the DB.

What are Telescope's limitations?
- Telescope does not provide a full-featured database query language.
- Telescope only supports a limited subset of Haskell data types.
- In a very early prototype stage, much is missing/untested!

Telescope is particularly well-suited for applications where events are pushed
by the server e.g. notifications and dashboards. Telescope also handles forms
and input-validation very well. On the flip-side, applications with heavy
client-side computation such as animations are not well-suited for Telescope.

## How it works!
- Write a `PrimaryKey` instance for your datatype to allow the `Entity` instance
  to be derived for your datatype via `Generics`.
- Telescope typeclass provides functions that operate on datatypes which are
  instances of `Entity`.
-  `Telescope` typeclass determines how to communicate with a database, you need
  to tell Telescope how to communicate with your DB (read/write these generic
  data types). Simple instance for demo purposes is included.
- Telescope exports a server which acts as a proxy to the database for any web
  clients, allowing web clients to execute the same Telescope library functions as
  if they were server-side!
- "Reactive" variants of the Telescope function to read/write data are provided,
  where function parameters and return values are streams of data (e.g. `Event
  Int`) as opposed to single values (e.g. `Int`).

## Try it out!
- Have [Nix](https://nixos.org/download.html) installed.
- Configure use of the Reflex-FRP cache, follow step 2
[here](https://github.com/obsidiansystems/obelisk#installing-obelisk).

``` bash
git clone --recurse-submodules https://github.com/jerbaroo/telescope
```

Commands for running the "chatroom" app in development mode:

``` bash
./scripts/run/dev.sh chatroom-backend
./scripts/run/dev.sh chatroom-frontend
# Then open localhost:3003 in a CHROMIUM browser.
./scripts/repl.sh    chatroom-backend
```

Commands for running the "chatroom" app in production mode:

``` bash 
./scripts/build/prod.sh chatroom-frontend
./scripts/build/prod.sh chatroom-backend
./scripts/run/prod.sh   chatroom-backend
# Then open localhost:3002 in a CHROMIUM browser.
```

Commands for hacking on the Telescope framework:

``` bash
./scripts/check.sh telescope
./scripts/test/suite.sh
./scripts/test/full.sh
./scripts/hoogle.sh 5000
```

## Telescope in 4 Steps
**1.** Define the data types used in your application.

``` haskell
data Message = Message
  { time     :: Int
  , room     :: Text
  , username :: Text
  , message  :: Text
  } deriving (Eq, Ord, Generic, Show)

instance PrimaryKey Message Int where
  primaryKey = time
```

**2.** Include Telescope's server in your backend code.

``` haskell
Server.run port id
```

**3.** Write your frontend with Reflex-DOM.

``` haskell
main = mainWidget $ do
  -- A text field to enter chat room name and username.
  (roomNameDyn, usernameDyn) <- do
    roomNameInput <- textInputPlaceholder "Chat Room"
    usernameInput <- textInputPlaceholder "Username"
    pure (roomNameInput ^. textInput_value, usernameInput ^. textInput_value)
  -- View messages live from the database.
  dbMessagesEvn <- T.viewTableRx $ const (Proxy @Message) <$> updated roomNameDyn
  -- Filter messages to the current chat room.
  roomMessagesDyn <- holdDyn [] $ attachPromptlyDynWith
    (\rn ms -> [m | m <- ms, room m == rn]) roomNameDyn dbMessagesEvn
  -- Display messages for the current chat room.
  el "ul" $ simpleList roomMessagesDyn $ el "li" . dynText . fmap
    (\m -> "“" <> username m <> "”: " <> message m)
  -- A text field for entering messages, and button to send the message.
  messageTextDyn <- (^. textInput_value) <$> textInputPlaceholder "Enter Message"
  timeEvn        <- fmap (fmap round) . tagTime =<< button "Send"
  -- Construct a 'Message' from user input, and send on button click.
  let messageToSendDyn = (\room username message time -> Message {..})
        <$> roomNameDyn <*> usernameDyn <*> messageTextDyn
      messageToSendEvn = attachPromptlyDynWith ($) messageToSendDyn timeEvn
  T.setRx messageToSendEvn
```

Finally build and run the application. You can now open the app in two browser
tabs, **interact with the app in one tab and watch the other app react**!

## Generic Representation

``` haskell
-- Example of a datatype to be stored.
data Person { name :: Text, age  :: Int } deriving Generic

instance PrimaryKey Person where
 primaryKey = name
  
-- Diagram showing conversion to/from storable representation.
Person "john" 70     <--->     "Person"
                               | ID     | "name" | "age" |
                               | "john" | "john" | 70    |
```
