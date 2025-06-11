---
title: Blackjack Data Structures & Algorithms
date: 2025-06-10 23:00:00 -0400
categories: [programming, DSA]
tags: [python, c++, typescript]
---

### Where I've Been the Last Couple of Weeks

For the past few weeks I have been learning Data Structures and Algorithms by watching a YouTube course and a <a href="https://frontendmasters.com/courses/algorithms/">free course on frontend masters</a>. It was challenging to get through the advanced data structures, so while I was still going through the courses, I decided to create a command line interface game of blackjack to implement some of the basic structures. I originally coded it in Python since that's the first language I was focused on and I got it working after about a week. But then the frontend masters course was using TypeScript as the example language and so as I was learning Trees, I ended up exploring learning other languages.

For anyone who is unfamilliar with programming languages, Python is known to be a very slow language when it comes to performance because it is interpreted line by line. While I was doing the course I learned more about TypeScript, which is just JavaScript with more enforced types. I didn't decide to go through a real tutorial for TypeScript (<b>TS</b>)/JavaScript (<b>JS</b>), because it is also a very slow and JavaScript has a bad reputation of being overused, needing a ton code to get the same logic, and being kind of stupid and people program it badly as well. So, I wanted to learn a *real* language that is performant, and I decided to learn <b>C++</b>. I have learned that the <b>C</b> languages are very powerful and fast because they are not incredibly high level. I intend to use <b>C++</b> as a platform to learn Golang (<b>Go</b>), which is another member of the <b>C</b> family and is used for web-servers instead of bad <b>JS</b> via Node.js. Then, I'll use HTMX to minimize the interactions with <b>JS</b> when I start to make a website for the game. So, I've been grinding to learn all of these concepts.

All of this shit talking on <b>JS</b> is all in fun jest. I will have to learn it to some extent if I am going to continue to program, because it's all over the internet, including this website template. I do have an interest in making web applications because I use web apps all of the time. So, I will properly learn some <b>TS</b> because it is more strongly typed and compiles to <b>JS</b>.

### The Underlying Data Structures for My Game

The game is made up of 2 data structures, 3 basic objects, and a lot of logic to interact with those structures in the game itself. The most basic element are the cards which make up a deck strucutre. The cards are nodes with suit, value, above, below and hidden attributes. The suit and value of a card are actual data, but the above and under are pointers to other cards which will be above and below it in the deck or to nothing if they are at the top or bottom of the deck or if the card is being removed. The hidden attribute is used to hide the suit and value for the dealer's second card. Here's the code for the card object.
#### Card Object
```python
class Card:
    """One single card. A node."""

    under = None
    above = None

    def __init__(self, suit, value, hidden=False):
        self.suit = suit
        self.value = value
        self.hidden = hidden

    def __repr__(self):
        return "[%s]" % (self.value + self.suit)
```
```c++
struct Card {
    // this structure is the actual memory for the Deck doubly-linked list.

    Card *under;
    Card *above;
    const std::string value;    // once I set a value, I can't change it
    const std::string suit;

    Card(std::string card_value, std::string card_suit): 
        value(card_value), suit(card_suit), under(nullptr), above(nullptr) {}

    ~Card() {std::cout << "Deleting card: " << this->print();}

    std::string print() {
        return "[" + value + suit + "]";
    }

};
```

The <b>C++</b> implementation doesn't have the hidden attribute because I didn't build out the whole game logic. Also significantly different is the Card destructor which is used in C languages because you have to manually control memory as there is no automated garbage collection. And finally is the strange use of `*`, that is used to specify that the variable is a pointer to a Card object (or data type if it wasn't a custom object).

The Deck is a real data structure which is a doubly-linked list has stack-like properties because we 'pop' off the top card to deal it and push cards to the top. But there's also a merge option to add a pile to the bottom of the deck which is used for shuffling the deck. The deck has pointers have pointers to the top and bottom cards and a card count. Here's the code in python.
#### Deck Structure
```python
class Deck:
    """
    A stack of playing cards which we can shuffle & deal. You can add more decks as well.
    Doubly-linked List. Slowest operation is wash shuffling O(n^2), quadratic. Dealing is O(1).
    Merging decks is also O(1). Splitting, Riffle shuffle, and Box shuffle are O(n) as you have
    to traverse portions of each deck to shuffle.
    """

    def __init__(self):
        self.numCards = 0
        self.top = None
        self.bottom = None

    def generateDeck(self):
        """Generate a deck in order A-2 of each suit."""

        values = ["2", "3", "4", "5", "6", "7", "8", "9", "T", "J", "Q", "K", "A"]
        suits = ["♠️", "♣️", "❤️", "♦️"]
        for suit in suits:
            for value in values:
                card = Card(suit, value)
                if self.numCards == 0:
                    self.numCards += 1
                    self.top = self.bottom = card
                else:
                    self.numCards += 1
                    card.under = self.top
                    self.top.above = card
                    self.top = card

    def deal(self):
        """Deal the top card of the deck."""

        self.numCards -= 1
        curr = self.top
        self.top = curr.under
        self.top.above = None
        curr.under = None
        return curr

    def add(self, card):
        """Add a card to the top of the deck."""

        self.numCards += 1
        if self.top is None:
            self.top = self.bottom = card
        else:
            self.top.above = card
            card.under = self.top
            self.top = card

    def mergeDecks(self, deck):
        """Put new deck below the current deck."""

        self.numCards += deck.numCards

        self.bottom.under = deck.top
        deck.top.above = self.bottom

        self.bottom = deck.bottom
        deck.top = None
        deck.bottom = None

        return self

    def split(self):
        """Split the deck exactly in half, returning the split decks."""

        botHalf = Deck()
        botHalf.bottom = self.bottom
        mid = self.numCards // 2

        curr = self.bottom
        for i in range(mid):
            self.numCards -= 1
            botHalf.numCards += 1
            curr = curr.above

        self.bottom = curr
        botHalf.top = curr.under
        curr.under = None
        botHalf.top.above = None

        return self, botHalf

    def wash(self):
        """
        Do a wash shuffle where we break all the connections between cards and reorder them.
        The approach used is to create a new deck by randomly selecting cards from the given deck.
        """

        d2 = Deck()
        # while the deck still has cards in it
        while self.numCards > 0:
            # choose a random card
            idx = random.randint(0, self.numCards - 1)
            if idx == 0:
                self.numCards -= 1
                tmp = self.top
                self.top = tmp.under
                tmp.under = None
                if self.top:
                    self.top.above = None
                d2.add(tmp)
            # iterate until we get to before the card and update pointers
            else:
                curr = self.top
                for i in range(self.numCards):
                    if i == (idx - 1):
                        self.numCards -= 1
                        tmp = curr.under
                        curr.under = tmp.under
                        tmp.under = None
                        if curr.under:
                            curr.under.above = None
                        d2.add(tmp)
                        break  # break the loop once we add the card the the new deck
                    else:
                        curr = curr.under
        # update the current deck to be the new deck by updating all of its attributes
        self.top = d2.top
        self.bottom = d2.bottom
        self.numCards = d2.numCards

        return self

    def riffleMix(self):
        self.numCards -= 1
        curr = self.bottom
        if self.bottom.above:
            self.bottom = self.bottom.above
            self.bottom.under = None
            curr.above = None
        return curr

    def riffle(self):
        """Classic shuffle by splitting the deck in half and interlocking the cards bottom up."""

        topHalf, botHalf = self.split()
        riffled = Deck()

        while topHalf.numCards or botHalf.numCards > 0:
            if topHalf.numCards > 0:
                riffled.add(topHalf.riffleMix())

            if botHalf.numCards > 0:
                riffled.add(botHalf.riffleMix())

        self.top = riffled.top
        self.bottom = riffled.bottom
        self.numCards = riffled.numCards

        return self

    def box(self):
        """
        Divide the deck in roughly 4ths and put them in a pile on top of each other.
        Divide 3 times by a random amount, then take the remainder and put it on top of the pile.
        """

        boxPile = Deck()
        fifth = self.numCards // 5
        third = self.numCards // 3

        for i in range(3):
            # Take a fraction of the deck each time
            # e.g., 1 deck, 52 cards, fifth = 10, third 17
            split = random.randint(fifth, third)
            curr = self.top
            self.numCards -= split
            boxPile.numCards += split

            for j in range(split - 1):
                curr = curr.under

            tmp = curr.under
            if boxPile.top:
                boxPile.top.above = curr  # point the old top before to be curr
                curr.under = boxPile.top  # point curr next card to be the old top
            boxPile.top = self.top
            self.top = tmp
            self.top.above = None
            if boxPile.bottom is None:
                boxPile.bottom = curr
                boxPile.bottom.under = None

        # with the last ~ 4th, just add it to the top of the pile
        boxPile.numCards += self.numCards
        boxPile.top.above = self.bottom
        self.bottom.under = boxPile.top
        boxPile.top = self.top
        self.bottom = boxPile.bottom
        self.top = boxPile.top
        self.numCards = boxPile.numCards

        return self

    def shuffle(self):
        """Do a 'Poker' shuffle of the deck. Wash the cards, then riffle 2x, then box, and finally riffle."""
        self.wash()
        self.riffle()
        self.riffle()
        self.box()
        self.riffle()
        return self

    def __repr__(self):
        nodes = []
        curr = self.top
        while curr:
            nodes.append("[%s]" % (curr.value + curr.suit))
            curr = curr.under
        return " <-> ".join(nodes)
```
The methods in <b>C++</b> are very similar, but you have to specify return types in the function declarations and with pointer objects, you don't have to return the object if you just update the pointers for the object you're working on. Then, you free up the memory for any `new` objects that were created.
```c++
struct Deck {

    /* This structure points to the memory for cards. It is based on a doubly-linked list.
    // It features push and pop for adding or removing  individual cards. Stack-like.
    // However, it also features a function to shuffle its pointers. 
    // As well as a function to add stacks of cards to the bottom.  */

    int card_count;
    Card *top;
    Card *bottom;

    Deck(): card_count(0), top(nullptr), bottom(nullptr) {}
    ~Deck() {
        this->top = nullptr;
        this->bottom = nullptr;
        delete this->top;
        delete this->bottom;
    }

    void generate() {

        std::string values[13] = {"2", "3", "4", "5", "6", "7", "8", "9", "T", "J", "Q", "K", "A"};
        std::string suits[4] = {"♠️", "♣️", "❤️", "♦️"};

        for (int i = 0; i < (sizeof(suits)/sizeof(suits[0])); i++) {
            for (int j = 0; j < (sizeof(values)/sizeof(values[0])); j++) {

                Card* new_card = new Card(values[j], suits[i]);
                if (this->card_count == 0) {
                    card_count++;
                    this->top = this-> bottom = new_card;
                } else {
                    card_count++;
                    new_card->under = this->top;
                    this->top->above = new_card;
                    this->top = new_card;
                }

            }
        }

    }

    Card* deal() {

        this->card_count--;
        Card* curr = this->top;

        this->top = curr->under;
        this->top->above = nullptr;
        curr->under = nullptr;

        return curr;

    }

    void push(Card *new_card) {
        // Update pointers to a card in memory to add the card to the top of this deck

        if (this->card_count == 0) {
            this->top = this->bottom = new_card;
        } else {
            this->top->above = new_card;
            new_card->under = this->top;
            this->top = new_card;
        }
        
        this->card_count++;

    }

    void merge_decks(Deck *bot_deck) {

        // add a deck (stack) of cards to the bottom of this deck assume that this deck is not empty
        this->card_count += bot_deck->card_count;

        // To update this deck, we have to point the current bottom card to the top of the new deck
        //  and update the bottom card pointer to the bottom of the new deck
        this->bottom->under = bot_deck->top;
        bot_deck->top->above = this->bottom;
        this->bottom = bot_deck->bottom;

    }

    Deck* split() {

        int mid = floor(this->card_count / 2);
        Deck* bot_half = new Deck();

        Card* curr = this->bottom;
        bot_half->bottom = this->bottom;

        for (int i = this->card_count; i > mid; i--) {

            this->card_count--;
            bot_half->card_count++;
            curr = curr->above;

        }

        this->bottom = curr;
        bot_half->top = curr->under;
        curr->under = nullptr;
        bot_half->top->above = nullptr;

        return bot_half;
    
    }

    void wash() {
        
        /* Choose a random card in this deck and put it into a temporary deck (deck2).
        // I want to pass the pointer to the memory location of the cards to not create new memory.        
        // Then, I want to reset the pointers for this deck to the memory allocations for the top 
        //  and bottom cards and revise the count.
        // The count will be used to make sure we don't lose track of the memory as we iterate. */

        // Set up the object with top & bottom pointers and an int attr of the number of cards
        Deck *deck2 = new Deck();
        while (this->card_count > 0) {

            // set up a random number generator based on the time this loop is called
            unsigned seed = std::chrono::system_clock::now().time_since_epoch().count();
            std::mt19937 generator(seed);
            std::uniform_int_distribution<int> distribution(0, this->card_count - 1);
            int idx = distribution(generator);

            Card *curr = this->top;
            // logic to quickly update pointers for the top so we can avoid list traversal
            // if we are pushing the top card. We use a pointer to follow the pointers of the object.
            if (idx == 0) {
                this->top = curr->under;
                if (this->top != nullptr) {this->top->above = nullptr;}
                curr->under = nullptr;
            } else {  // else we do a traversal to the card
                // list traversal from top to the card we chose.
                for (int i = 0; i < idx; i++) {curr = curr->under;}

                // remove this deck's pointers to the memory of the card we chose.
                if (curr->above != nullptr) {curr->above->under = curr->under;}
                if (curr->under != nullptr) {curr->under->above = curr->above;}
                curr->above = nullptr;
                curr->under = nullptr;
            }
            // take the current card's memory location and add it to the temp deck
            // decrease this deck's count for the next loop
            deck2->push(curr);
            this->card_count--;

        }

        // make the now empty this deck the temp deck
        this->top = deck2->top;
        this->bottom = deck2->bottom;
        this->card_count = deck2->card_count;

        // the deck we originally had should essentially be modified in place
        // so we can return void and use the same object we had before
        delete deck2;

    }

    Card *riffle_mix() {
        // return the bottom card of the deck that was used to call the function
        this->card_count--;
        Card *bot_card = this->bottom;
        if (bot_card->above != nullptr) {
            this->bottom = bot_card->above;
            this->bottom->under = nullptr;
            bot_card->above = nullptr;
        }

        return bot_card;

    }

    void riffle() {
        // split the current deck in half and then mix them into a new deck
        Deck *bot_half = this->split();
        Deck *riffled = new Deck();

        while (this->card_count > 0 || bot_half->card_count > 0) {
            if (this->card_count > 0) {riffled->push(this->riffle_mix());}
            if (bot_half->card_count > 0) {riffled->push(bot_half->riffle_mix());}
        }

        this->top = riffled->top;
        this->bottom = riffled->bottom;
        this->card_count = riffled->card_count;

        delete bot_half;
        delete riffled;

    }

    void box() {

        Deck *pile = new Deck();
        int fifth = floor(this->card_count / 5);
        int third = floor(this->card_count / 3);

        for (int i = 0; i < 3; i++) { // roughly split the deck into 4ths, grabbing between a fifth and a third of the deck
            // set up a random number generator based on the time this loop is called
            // choosing a number between a 3rd/5th of the deck
            unsigned seed = std::chrono::system_clock::now().time_since_epoch().count();
            std::mt19937 generator(seed);
            std::uniform_int_distribution<int> distribution(fifth, third);
            int idx = distribution(generator);

            this->card_count -= idx;
            pile->card_count += idx;
            // Travel to the card that we want to split at
            Card *curr = this->top;
            for (int j = 0; j < idx; j++) {curr = curr->under;}
            // keep a pointer to the tmp card which will be the top of the deck after we take off this stack
            Card *new_top = curr->under;
            // when the pile exists, connect the top of it to the current card
            if (pile->top != nullptr) {
                pile->top->above = curr;
                curr->under = pile->top;
            }
            // split the cards we grabbed from the rest of the deck and put it on the pile.
            pile->top = this->top;
            // if the pile didn't exist
            if (pile->bottom == nullptr) {
                pile->bottom = curr;
                curr->under = nullptr;
            }
            this->top = new_top;
            new_top->above = nullptr;

        }

        // the final fourth can be placed on the pile.
        pile->card_count += this->card_count;
        pile->top->above = this->bottom;
        this->bottom->under = pile->top;
        pile->top = this->top;
        // then just make this deck be the pile
        this->top = pile->top;
        this->bottom = pile->bottom;
        this->card_count = pile->card_count;

        delete pile;

    }

    void shuffle() {
        this->wash();
        this->riffle();
        this->riffle();
        this->box();
        this->riffle();
    }

    void print() {

        if (this->card_count == 0) {std::cout << "Empty deck.";}
        else {
            Card *curr = this->top;
            while (curr->under != nullptr) {
                std::cout << curr->print() << ", ";
                curr = curr->under;
            }
            std::cout << curr->print() << "\n";
        }

    }

};
```

### Player & Hand Objects
Sorry for the code dumps, hopefully you can use the contents to skip around. For the longer code I'll only summarize now that there's proof that I've coded over 1000 lines for this project. The next objects we will discuss are the Player and the Hand objects. A player is another Node with pointers to the next and prev players, a dealer flag, a player number, and a pointer to a hand object.
```python
class Player:
    """Create a player with a number designation,
    pointers to who comes before and after, and a dealer flag."""

    def __init__(self, number):
        self.next = None  # player object
        self.prev = None  # player object
        self.hand = Hand()  # hand object is a list of card objects
        if number == 1:
            self.dealer = True
            self.number = number
        else:
            self.dealer = False
            self.number = number

    def __repr__(self):
        if self.dealer:
            return "[🤖|P%s]" % self.number
        return "[🤡|P%s]" % self.number
```
The hand object is an object which holds cards in an array and keeps track of the number of cards that it has. There are methods to modify the cards in the hands.
```python
class Hand:
    """The player's hand of cards. A pylist of cards."""

    def __init__(self):
        self.cards = []
        self.size = 0

    def add(self, card):
        self.size += 1
        newCard = Card(card.suit, card.value)
        self.cards.append(newCard)
        return self.cards

    def addHidden(self, card):
        self.size += 1
        newCard = Card(card.suit, card.value, True)
        self.cards.append(newCard)
        return self.cards

    def reveal(self):
        self.cards[1].hidden = False
        return self.cards

    def remove(self):
        discard = Deck()

        for i in range(self.size):
            curr = self.cards[i]
            discard.add(curr)

        self.cards = []
        self.size = 0

        return discard

    def __repr__(self):
        nodes = []
        for card in self.cards:
            if card.hidden:
                nodes.append("[??]")
            else:
                nodes.append(str(card))
        return "".join(nodes)
```
### Players Ring List
Finally, the 2nd data structure is a ring list of players. A ring list is a structure with nodes which loop back around that way that after all the players are done in a turn, you can loop around and do things like remove the cards from the hand or keep looping around. There is still a semblance of who is the first and last player, but the nodes never point to the ether unless the player is being removed from the game.

```python
class Players:
    """Create the ring buffer so that the game can progress."""

    def __init__(self):
        self.first = None
        self.last = None
        self.numPlayers = 0

    def addPlayer(self, number):
        """Add a player to the end of the ring, looping around."""

        self.numPlayers += 1
        newPlayer = Player(number)

        if newPlayer.number == 1:
            self.first = self.last = newPlayer
            return

        newPlayer.next = self.first
        newPlayer.prev = self.last
        self.last.next = newPlayer
        self.first.prev = newPlayer
        self.last = newPlayer

    def insertPlayer(self, number):
        """
        Insert a player and move the other players down a 'seat', retaining their hands.
        Inserted player's hand is empty
        """

        num = self.numPlayers + 1
        self.addPlayer(num)

        curr = self.last
        while curr.number > number:
            curr.hand = curr.prev.hand
            curr = curr.prev

        curr.next.hand = Hand()

    def __repr__(self):
        curr = self.first
        count = 0
        players = []

        while count < self.numPlayers:
            count += 1
            if curr.dealer:
                players.append("[🤖|P%s]" % curr.number)
                curr = curr.next
            else:
                players.append("[🤡|P%s]" % curr.number)
                curr = curr.next

        return " -> ".join(players)
```
### Closing
That's all for this post. I've dumped way to much code for this to be easily readable, but the deed is done. I've finished learning the data structures, but I need to practice the advanced structures and practice recursion. The game works, so while I'm practicing those, I'm going to try to code a server to run the game and a frontend to make it not be a commandline game. I'll see if my sister can make the assets for cards and stuff. Oh and of course I need to post about the game's logic. Cya nerds bye!