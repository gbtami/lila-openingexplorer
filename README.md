Opening explorer for lichess.org
================================

[![Build Status](https://travis-ci.org/niklasf/lila-openingexplorer.svg?branch=master)](https://travis-ci.org/niklasf/lila-openingexplorer)

Preparations
------------

Assuming `build-essential`, `openjdk-8-jdk`, `scala` and `sbt` are installed.
You already have this, if you are running a local lila instance.

    ./bin/build-deps.sh  # install scalalib

Install Kyoto cabinet headers and utilities.

    sudo apt-get install libkyotocabinet-dev kyotocabinet-utils

Setup the `$JAVA_HOME` environment variable.

    # on Debian:
    export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64

    # or, on ArchLinux:
    export JAVA_HOME=/usr/lib/jvm/java-8-openjdk

Download and unpack [Kyotocabinet Java package](http://fallabs.com/kyotocabinet/javapkg/).

    curl http://fallabs.com/kyotocabinet/javapkg/kyotocabinet-java-1.24.tar.gz | tar xvz
    cd kyotocabinet-java-1.24
    ./configure
    make
    sudo make install

Create configuration file.

    cp conf/application.conf.example conf/application.conf

Run server
----------

    sbt run -Djava.library.path=/usr/local/lib/

Index master games
------------------

    python3 bin/index-pgn.py master <games.pgn>

HTTP API
--------

CORS enabled for all domains. Provide `callback` parameter to use JSONP.

### `GET /master` query opening database with master games

```
> curl http://expl.lichess.org/master?fen=rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR%20w%20KQkq%20-%200%201
```

name | type | default | description
--- | --- | --- | ---
**fen** | string | required | FEN of the position to look up
**moves** | int | 12 | number of most common moves to display
**topGames** | int | 4 | number of top games to display. maximum is 4

```javascript
{
  "fen": "rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR w KQkq - 0 1",
  "white": 645088,
  "draws": 838971,
  "black": 462332,
  "averageRating": 2407,
  "moves": [  // sorted by total number of games, higher first
    {
      "uci": "e2e4",
      "san": "e4",
      "white": 291056,
      "draws": 376163,
      "black": 218378,
      "averageRating": 2401
    },
    {
      "uci": "d2d4",
      "san": "d4",
      "white": 234074,
      "draws": 304134,
      "black": 159442,
      "averageRating": 2414
    },
    // ...
  ],
  "topGames": [  // higher ratings first
    {
      "id": "JzjKnFKm",
      "winner": "white",
      "white": {
        "name": "Aronian, L.",
        "rating": 2826
      },
      "black": {
        "name": "Carlsen, M.",
        "rating": 2872
      },
      "year": 2014
    },
    // ...
  ],
  "recentGames": []  // roughly ordered by date, newer games first
}
```

### `GET /master/pgn/{id}` fetch one master game by ID

```
> curl http://expl.lichess.org/master/pgn/6UKt001Q
```

```
[Event "Wch Blitz"]
[Site "Astana"]
[Date "2012.07.10"]
[Round "23"]
[White "Carlsen, Magnus"]
[Black "Chadaev, Nikolay"]
[Result "1-0"]
[WhiteElo "2837"]
[BlackElo "2580"]

1. e4 e5 2. f4 d5 3. exd5 exf4 4. Nf3 Nf6 5. c4 c6 6. d4 cxd5 7. c5 Nc6 8. Bb5 Be7 9. O-O O-O 10. Bxf4 Bg4 11. Nc3 Ne4 12. Qd3 Bf5 13. Qe3 Bf6 14. Bxc6 bxc6 15. Ne5 Bxe5 16. Bxe5 Bg6 17. Nxe4 Bxe4 18. Qg3 f6 19. Bd6 Re8 20. b4 Bg6 21. a4 a6 22. h4 Qd7 23. h5 Bxh5 24. Rxf6 Qg4 25. Qxg4 Bxg4 26. Rf4 Bh5 27. Raf1 h6 28. Be5 Ra7 29. b5 axb5 30. axb5 cxb5 31. c6 Raa8 32. c7 Kh7 33. Rb1 Be2 34. Rf7 Rg8 35. Re7 Bc4 36. Kh2 Rae8 37. Rd7 Ra8 38. Rb2 Raf8 39. g4 Ra8 40. Rf2 b4 41. Rff7 h5 42. Rxg7+ Rxg7 43. Rxg7+ 1-0
```

### `GET /lichess` query lichess opening database

```
> curl http://expl.lichess.org/lichess?variant=standard&speeds[]=blitz&speeds[]=classical&ratings[]=2200&ratings[]=2500&fen=rnbqkbnr/pppppppp/8/8/8/8/PPPPPPPP/RNBQKBNR%20w%20KQkq%20-%200%201
```

name | type | default | description
--- | --- | --- | ---
**fen** | string | required | FEN of the position to look up
**variant** | string | required | one of `standard`, `antichess`, `chess960`, `horde`, `racingKings`, `threeCheck`, `atomic`, `crazyhouse` or `kingOfTheHill`
**speeds[]** | list | none | `bullet`, `blitz` and/or `classical`
**ratings[]** | list | none | rating groups ranging from their value to the next higher group: `1600`, `1800`, `2000`, `2200` and `2500` to unlimited
**moves** | int | 12 | number of most common moves to display
**topGames** | int | 4 | number of top games to display. maximum is 4
**recentGames** | int | 4 | number of recent games to display. many may be available


### `GET /stats` get database stats

```
> curl http://expl.lichess.org/stats
```

```javascript
{
  "master": {
    "games": 1924132,
    "uniquePositions": 49260139
  },
  "lichess": {
    "standard": {
      "games": 10361404,
      "uniquePositions": 335956395
    },
    "chess960": {
      "games": 264960,
      "uniquePositions": 14469620
    },
    "crazyhouse": {
      "games": 134221,
      "uniquePositions": 4512974
    },
    // other variants ...
  }
}
```
