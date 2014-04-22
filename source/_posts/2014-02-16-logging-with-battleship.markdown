---
layout: post
title: "Logging with Battleship"
date: 2014-02-16 09:19:22 -0400
comments: true
categories: ruby
---
After the first week at flatiron School, I decided to ~~punish~~ challenge myself by joining the [Ruby Fight Club](http://www.meetup.com/Ruby-Fight-Club/) meetup and signing myself up for the Battleship competition.

The task at hand is to code a simple player class that plays battleship by doing two things:

1. Placing its ships at the beginning of a game
2. Given the state of the game board and the remaining ships, making a move by returning a set of x,y coordinates. 

My strategy loosely follows the [one posted by Nick Berry](http://www.datagenetics.com/blog/december32011/index.html). While I’ll save the details for another post, the strategy has two basic modes:

- Search mode – No ship locations are known.
- Hunt mode – There has been a hit but the ship hasn’t been sunk.

In search mode, the algorithm tries to calculate the most likely places for a ship to have been hidden, using the known information about missed shots and remaining ships. In hunt mode, a ship has been hit and the algorithm looks around the successful hit until the ship has been sunk.

At first I tried my best to be a good little Ruby student and go the test/behavior driven development route. This strategy worked well for setting up many of the early helper methods and even test some of the basic early game strategy. For instance, in the beginning of the game, the four cells in the center have the highest probability of having a ship, so the get_best_moves method should always include [4,4]. Rspec makes testing this ridiculously simple:

```ruby
before(:each) do
  @eugmill = EugmillPlayer.new()
end

describe "#get_best_moves" do
  it "should include center coordinates for a blank board" do
    expect(@eugmill.get_best_moves(board1,[2,3,3,4,5])).to include([4,4]) 
  end
end
```

##Bigger problems

Testing with Rspec worked beautifully for these early game scenarios, when our Player object was still innocent and the board untouched by the violence of naval warfare. Difficulty arises when testing for and troubleshooting scenarios 20 moves into the game.

Enter the logger class. Ruby’s built-in logger lets you write to files to keep track of what happens in your application. This way, when your player class makes a bizarre move halfway into the game, you can see exactly what was going on inside your class, as well as the list of moves leading up to that move. Here is an example of one of my log files:

```ruby
I, [2014-02-16T14:27:05.225023 #89817]  INFO -- board states: 
   0  1  2  3  4  5  6  7  8  9
0 [ ][ ][ ][ ][ ][ ][ ][ ][ ][ ]
1 [ ][ ][ ][ ][M][ ][ ][ ][ ][ ]
2 [ ][ ][H][ ][ ][ ][M][ ][ ][ ]
3 [ ][ ][ ][M][ ][ ][ ][M][ ][ ]
4 [ ][H][ ][ ][ ][M][ ][ ][ ][ ]
5 [ ][M][ ][ ][M][ ][ ][ ][M][ ]
6 [ ][ ][M][ ][ ][ ][M][ ][ ][ ]
7 [ ][ ][ ][M][ ][ ][ ][M][ ][ ]
8 [ ][ ][ ][ ][ ][M][ ][ ][ ][ ]
9 [ ][ ][ ][ ][ ][ ][ ][ ][ ][ ]

I, [2014-02-16T14:27:05.225136 #89817]  INFO -- search mode: Move 16,Getting best move...
I, [2014-02-16T14:27:05.233883 #89817]  INFO -- game state: Making move 4,0
I, [2014-02-16T14:27:05.631970 #89817]  INFO -- hit: It was a hit!!!
I, [2014-02-16T14:27:05.632115 #89817]  INFO -- hit: [[4, 1], [2, 2], [4, 0]]
I, [2014-02-16T14:27:05.632235 #89817]  INFO -- hit: Ship sunk!size:2
I, [2014-02-16T14:27:05.632408 #89817]  INFO -- hit: Sunk ship coords: [[4, 0], [4, 1]]
I, [2014-02-16T14:27:05.633557 #89817]  INFO -- board states: 
   0  1  2  3  4  5  6  7  8  9
0 [ ][ ][ ][ ][ ][ ][ ][ ][ ][ ]
1 [ ][ ][ ][ ][M][ ][ ][ ][ ][ ]
2 [ ][ ][H][ ][ ][ ][M][ ][ ][ ]
3 [ ][ ][ ][M][ ][ ][ ][M][ ][ ]
4 [H][H][ ][ ][ ][M][ ][ ][ ][ ]
5 [ ][M][ ][ ][M][ ][ ][ ][M][ ]
6 [ ][ ][M][ ][ ][ ][M][ ][ ][ ]
7 [ ][ ][ ][M][ ][ ][ ][M][ ][ ]
8 [ ][ ][ ][ ][ ][M][ ][ ][ ][ ]
9 [ ][ ][ ][ ][ ][ ][ ][ ][ ][ ] 
```
From this we know that on move 16, it takes a shot at [4,0]. We can do this for any move in the game. Now if we’re curious about why it made that move, we can also pull up our second log file which logs the probability distributions for the board at each move:

```ruby
I, [2014-02-16T14:27:05.233150 #89817]  INFO -- pboard: Board for move 16
I, [2014-02-16T14:27:05.233301 #89817]  INFO -- pboard: 
[10, 15, 19, 19, 17, 21, 17, 17, 15, 10]
[14, 16, 17, 8 , 0 , 12, 10, 15, 18, 15]
[19, 21, 26, 16, 13, 12, 0 , 6 , 15, 17]
[19, 13, 16, 0 , 7 , 8 , 6 , 0 , 10, 17]
[22, 14, 21, 12, 8 , 0 , 8 , 10, 12, 21]
[17, 0 , 6 , 5 , 0 , 6 , 7 , 7 , 0 , 17]
[17, 5 , 0 , 6 , 8 , 7 , 0 , 6 , 8 , 19]
[17, 11, 6 , 0 , 10, 7 , 6 , 0 , 8 , 15]
[15, 16, 15, 10, 12, 0 , 8 , 8 , 14, 14]
[10, 14, 17, 17, 21, 17, 19, 15, 14, 10]
```
Skipping over how the probability weights are calculated, we can see that cell [4,0] has a weight of 22, which is the maximum for this board, and also the reason it was targeted.

So how do we set up our logging? In this case, we initialize our loggers whenever a new Player object is created:

```ruby
class EugmillPlayer
  def initialize
    @movelogger = Logger.new("./players/logs/mylog.txt")
    @boardlogger= Logger.new("./players/logs/board_log.txt")
    ...
  end
  ...
end
```

Then, whenever we want to log a piece of information, we can call our logger

```ruby
@logger.info('board states'){pprint_board(state)}
  if @target_mode == :search
    @logger.info("search mode") {"Move #{@move_counter},Getting best move..."}
    moves = get_best_moves(state,ships_remaining,pboard)
    row,col = moves[rand(moves.size)]
    @logger.info('game state') {"Making move #{row},#{col}"}
...
```
The logger has different severity levels, depending on whether we’re logging an error or just debugging information. There are multiple ways to pass messages to our logger. In this case I am passing a label as a string parameter, and a the bulk the message in a block. The result looks like the screenshot posted above.

From a technical perspective, there isn’t anything difficult about the logger class. If the logging is too terse, debugging is difficult, and if it is too verbose, the logs grow out of control pretty quickly. Finding the right mix makes logger indespensible to debugging complex scenarios quickly.

Ultimately, I found logger to be complementary to Rspec, rather than a replacement.  With Logger I can catch strange behavior and try to reproduce it, while with Rspec I can make sure to test for and prevent the same errors from happening again. 