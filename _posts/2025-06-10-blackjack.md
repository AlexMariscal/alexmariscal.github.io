---
title: Blackjack Data Structures & Algorithms
date: 2025-06-10 23:00:00 -0400
categories: [programming, DSA]
tags: [python, c++, typescript]
math: true
---

### Where I've Been the Last Couple of Weeks

For the past few weeks I have been learning Data Structures and Algorithms by watching a YouTube course and a <a href="https://frontendmasters.com/courses/algorithms/">free course on frontend masters</a>. It was challenging to get through the advanced data structures, so while I was still going through the courses, I decided to create a command line interface game of blackjack to implement some of the basic structures. I originally coded it in Python since that's the first language I was focused on and I got it working after about a week. But then the frontend masters course was using TypeScript as the example language and so as I was learning Trees, I started exploring learning other languages.

For anyone who is unfamilliar with programming languages, Python is known to be a very slow language when it comes to performance because it is interpreted line by line. While I was doing the course I learned more about TypeScript, which is just JavaScript with stricter typing. JavaScript is ubiquitos because it's the language your internet browser uses to update webpages without full page reloads, allowing for cool dynamic websites. However, I decided against trying a real tutorial for TypeScript (<b>TS</b>)/JavaScript (<b>JS</b>), because it is also a very slow and JavaScript has a bad reputation of being overused, needing a ton code to implement logic, and being kind of generally janky. So, I wanted to learn a *real* language that is performant, and I decided to learn a bit of <b>C++</b>. I decided this because I learned that the <b>C</b> languages are very powerful and fast because they aren't highly abstract and give you a lot of control over the memory that you use for programming. But, because of the ammount of power they give you, they can be difficult to use when you're free to "shoot yourself in the foot". I intend to use <b>C++</b> as a platform to learn how to properly manage memory, so I understand when I'm being wasteful, and to understand how to really make programs fast. I will also use it to have an introduction to <b>C</b> languages because I want to learn Golang (<b>Go</b>), because I've heard it is simple and very good for web-servers instead of having to use <b>JS</b>. And finally, to avoid <b>JS</b> as much as possible, I'll learn HTMX to make the frontend of websites. So, I've been grinding to learn a lot of background so I can really take off with all of this coding stuff.

All of this shit talking on <b>JS</b> is not actually incredibly serious and snobby, it is a bit of a meme to call <b>JS</b> bad. I will have to learn it if I am going to continue to program because it's why the internet is now able to be full of cool, dynamic websites, including this website template. I do have an interest in making web applications because I use web apps all of the time. So, I will properly learn some <b>TS</b> because it is more strongly typed and compiles to <b>JS</b>.

### Data Structures Organize Data Objects

So let's get back to data structures! Data structures are what the name implies, they are ways to organize data that is useful for programming. I kind of like card games and blackjack seemed to be the easiest one to implement. I have further ideas to put the game on a website so that you can play for free and learn how to count cards instead of having to gamble real money on seedy websites. In order to play the game after writing it on the computer, you need to create data objects and organize them in structures. I approached the game by coding the cards, a deck structure for the cards, and then players, player's hands which hold cards, and a structure for the players to control whose turn it is and to continue to the next player.

After about 1 week I got the implementation to work using Python, allowing me to play up to 8 hands at a time versus an automated dealer and play until prompted if I want to quit. In this post, I will only go over the data, data structures, and shuffling algorithms that I made for the game logic to use. In a follow up post, I will go over the code in the game itself and go over the logic and algorithms to get that working.

### Card Object

The most basic element for card games are the cards which are organized in a deck strucutre. The cards in my implementation are nodes with suit, value, above, below and hidden attributes. A node is just a data point that have pointers to other nodes which structures use to build the overall "shape" of the structure. The hidden attribute is something used by the game logic to hide the dealers second card, so I won't get into it. The suit and value of a card are actual data stored in memory recording the characters like the 2 (value) of ♣️ (suit). Then, the above and under attributes are pointers to other cards or to nothing depending on the situation. For example, if a card is being delt from the top of the deck to a player, you have to remove it from the deck, so the card can't point to other cards before it's put into the players hand. Otherwise we'd expect it to be in the deck, usually somewhere in the middle, so the program needs to know where it is relative to other cards to keep the structure intact. Let's look at the whole "Class" or blueprint for the cards in Python and C++.

```python
class Card:
    """One single card. A node."""

    under = None
    above = None

    def __init__(self, card_suit, card_value, hide = False):
        self.suit = card_suit
        self.value = card_value
        self.hidden = hide

    def __repr__(self):
        return "[%s]" % (self.value + self.suit)
```

The <b>C++</b> implementation probably immediately looks different because you have to define types for all variables and define the types for the values that functions return. Also, there's the strange use of `*`, that is used to specify that the variable is a pointer to a specific type of value. Since I've mentioned pointers before, let's give a crude definition: a pointer is a variable whose data is the memory address for another bit of data, so it allows us to create memory and keep it in separate places instead of being limited to creating a bunch of memory all at once. Another difference is the Card destructor (~Card) which is used in C languages because you have to manually control memory as there is no automated garbage collection. As a final note, the `hide = false` is a default argument which is just a way to say, if he function is not passed the hide argument, default it to false.

```c++
struct Card {
    // this structure is the actual memory for the Deck doubly-linked list.

    Card *under;
    Card *above;
    const std::string value;    // const: once I set a value, I can't change it
    const std::string suit;
    bool hidden;

    Card(std::string card_value, std::string card_suit, bool hide = false): 
        value(card_value), suit(card_suit), under(nullptr), above(nullptr), hidden(hide) {}

    ~Card() {std::cout << "Deleting card: " << this->print();}

    std::string print() {return "[" + value + suit + "]";}

};
```

### Deck Structure

The structure of the deck is a stack of cards. This concept is understood in computer science and is characterized by having a first-in-last-out structure, meaning that you can only access the top of the stack. You may be thinking, well what about when you shuffle the deck like in real-life, and you're right the structure ends up being more complicated than just a regular stack. Under the hood, the deck is a doubly-linked list because each card has pointers in 2 directions. Now that I look back, I am not sure that I need it to be doubly-linked, but it made it conceptually easy to remove cards from the deck from any position and add them to the top or bottom of the deck. The deck has pointers have pointers to the top and bottom cards and a card count. The regular stack methods which we have are to "pop" (deal) off the top card to deal it and "push" (add) cards to the top of the deck. But then I also coded a merge method to add a pile of cards to the bottom of the deck which is used for shuffling the deck. Shuffling the deck includes 3 algorithms to randomly order the cards.

#### Constructing the Deck

Let's start with the deck constructor, where we initialize an empty deck, and the deck generation, which fills the deck with 52 cards A-2 for each suit. The reason I initialize the deck as being empty is so that I can create empty deck objects which I can then fill with cards in an existing deck so that I can create temporary stacks of cards during a shuffle. Cards are generated by looping over the 13 values for cards and the 4 suits to generate the 52 unique card combinations. As a note, <b>C++</b> is much better at handling loops than Python, so here we have the first major instance of the <b>C++</b> program being faster.

We add each new card to the top of the deck. Inside the nested loops is code which first creates a card with the current value and suit, starting at the 2♠️. Then, we have a quick logic check to see if this is the first card to be added. If it is (if the deck's count is 0), you only have to point the deck's top and bottom at the new card, the card's pointers don't need to be updated to point at anything. To add a card to the deck when one exists, you have to update the new card's "under" pointer to the current top of the deck, the current "top" of the deck's pointer to the new card, and the update deck's "top" pointer and point to the new card, in that order so you don't lose pointers to keep track of each card. And finally, you increase the card count.

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

        suits = ["♠️", "♣️", "❤️", "♦️"]
        values = ["2", "3", "4", "5", "6", "7", "8", "9", "T", "J", "Q", "K", "A"]

        for suit in suits:
            for value in values:

                new_card = Card(suit, value)

                if self.numCards == 0:
                    self.top = self.bottom = new_card
                else:
                    new_card.under = self.top
                    self.top.above = new_card
                    self.top = new_card
                
                self.numCards += 1
```

Here's the implementation in <b>C++</b>. There is a ~Deck destructor method which points a deck's pointers to nothing to prevent "hanging pointers" and then deletes them. The card_count is a regular variable, so once it's out of scope it should be deleted without a fuss, as far as I understand.

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

        std::string suits[4] = {"♠️", "♣️", "❤️", "♦️"};
        std::string values[13] = {"2", "3", "4", "5", "6", "7", "8", "9", "T", "J", "Q", "K", "A"};

        for (int i = 0; i < (sizeof(suits)/sizeof(suits[0])); i++) {
            for (int j = 0; j < (sizeof(values)/sizeof(values[0])); j++) {

                Card* new_card = new Card(values[j], suits[i]);

                if (this->card_count == 0) {
                    this->top = this-> bottom = new_card;
                } else {
                    new_card->under = this->top;
                    this->top->above = new_card;
                    this->top = new_card;
                }

                card_count++;

            }
        }

    }
```

#### Dealing, Adding, Merging Decks, and Splitting a Deck
Next, let's look at some methods for the deck which I think extend to multiple card games. In every card game you deal a card from the top of the deck to a player, add individual cards to a deck (or small stack), have to merge decks via cuts, and split the deck in half. Let's go over the add (push) and deal (pop) functions associated with the property of a stack object.

These push and pop operations are really fast operations because you only have to update pointers of the top cards. What really takes time is trying to go through the deck to find a card at a specific index, which is often refered to as "list traversal", because its runtime grows based on the size of the input (the number of cards in the deck). The deal and add functions always take the same amount of time, no matter how large the deck is. For the add function a new pointer is defined, pointing at the top card so that we can give that pointer to whoever called the function. You have to break the pointer connections to/from this card and decrement the size of the deck. The add function takes in a card object which we point the top of the deck to and point the card at the deck, then update the top card and increment the deck size.

The merge deck function allows the deck to grow, potentially allowing you to play with multiple 52 card decks. It's also used to shuffle decks together as we will see later. This operation is also very fast since we only have to manipulate pointers for 2 cards again. We get a new deck object as an input and add the number of cards in it to the current deck. Then we update pointers so that the top of the new deck gets put to the bottom of this deck and the bottom pointer of the current deck to the bottom of the new deck, effectively making one merged deck. We don't have to return anything as the current deck is modified in place and let the deck that was being added be deleted.

The split function manipulates the current deck and splits it in half, returning a bottom half of the deck and we still have the top half of the current deck. To do the split, we start from the bottom card and then iteratively move to the calculated midpoint. As we iterate, we note how many cards have been set aside for the bottom deck and take away that count from the top deck. Once we get to the card, we update pointers so that the current card is the bottom of the current deck and the card under is the top of the bottom half. Doing the list traversal makes this method's runtime grow linearly with the size of the deck input.

```python
# Part of `class Deck:` from above
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

        return botHalf
```
In <b>C++</b>:

```c++
// part of `struct Deck {` from above
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

        // dividing ints won't return a float, no need for floor function call
        int mid = this->card_count / 2;
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
```
#### Shuffling the Deck
Finally, we have some algorithm methods to shuffle the deck. It should not be easy to predict the outcome of the shuffled deck, so we have to introduce some randomness. This is done by using random number generation libraries built-in to each language to pick numbers between the amount of cards that we specify. I have 3 separate shuffle methods which are 3 methods used in casinos to shuffle cards. 

The first is the most random and slowest method, the wash. The behavior we're trying to program is when a dealer spreads out a whole deck on the table to mix them. Ideally, this completely breaks every connection that each card has and reorders the whole deck. Then is the riffle, which is what I think of when thinking about shuffling cards, you split the deck in half and alternate which card goes into a pile as you fan through the cards. Finally is the box, where you take roughly a quarter of the deck from the top and put it in a pile and stack the next fourth on top of that pile. I then put them together in the order wash, riffle, riffle,box, riffle to simulate how poker is shuffled.

Here's a video to demonstrate the behavior that I tried to program: <a href="https://youtu.be/UeEMaZqMRp0?si=KuziFM4eQw9ivss1&t=373">https://youtu.be/UeEMaZqMRp0?si=KuziFM4eQw9ivss1&t=373</a>

There is a general issue with the deck shuffling and that is the amount of traversals that need to happen. As I mentioned, getting to a card in the middle of a deck is a slow operation which we have to do here because I don't have a system to index the cards before/during/after shuffling. The way I coded the wash method is especially slow because I randomly choose a card in the current deck and decide to put it in the new deck. In the worst case, this is a quadratic runtime, which means I am basically doing the square of the cards in the deck operations. If there are 52 cards and we have to go to the last card in the deck every time we choose a card, I have to point to the top and then travel 51 times, then point to the top and travel 50 times, then 49, etc. That works out to be 1378 operations.

You get this number because you have 26 pairs of numbers between 1 and 52 (52 is the max number of operations to do), and the sum of each pair is 53. Gauss proved that the sum of a series of consecutive numbers is equal to 

$$
\frac{n * (n+1)}{2} = \frac{n^{2}}{2} + \frac{n}{2}
$$

which is how you get $$ \frac{52*(53)}{2} = 1378 $$ operations. When you are considering runtime, you drop the constants and non-significant numbers so computer scientists say this is O(n<sup>2</sup>) runtime. In practice running my program still happens pretty much instantly because computers are so fast doing billions of operations per second. I recently learned that to make this faster, my nodes should be stored in a hash-map data structure and then I wouldn't have to code list traversals as it would be only 1 operation to go to a specific card, but that will be some other time.

```python
# Part of `class Deck:` from above
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

        botHalf = self.split()
        riffled = Deck()

        while self.numCards or botHalf.numCards > 0:
            if self.numCards > 0:
                riffled.add(self.riffleMix())

            if botHalf.numCards > 0:
                riffled.add(botHalf.riffleMix())

        self.top = riffled.top
        self.bottom = riffled.bottom
        self.numCards = riffled.numCards

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

    def shuffle(self):
        """Do a 'Poker' shuffle of the deck. Wash the cards, then riffle 2x, then box, and finally riffle."""
        self.wash()
        self.riffle()
        self.riffle()
        self.box()
        self.riffle()

    def __repr__(self):
        nodes = []
        curr = self.top
        while curr:
            nodes.append("[%s]" % (curr.value + curr.suit))
            curr = curr.under
        return " <-> ".join(nodes)
```

Here's the <b>C++</b> code. I noted it more thoroughly because I wanted to make sure I understood everything. In the shuffling implementations, we make temporary deck objects, so I made sure to delete them as needed.

```c++

    void wash() {
        
        /* Choose a random card in this deck and put it into a temporary deck (deck2).     
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
Sorry for the code dumps, hopefully you can use the contents to skip around. For the longer code I'll only summarize now that there's proof that I've coded over 1000 lines for this project. The next objects we will discuss are the Player and the Hand objects. A player is another Node with pointers to the next and prev players, a dealer flag, a player number, and a pointer to a hand object. The first player is called the dealer, and is automated in the blackjack program based on this flag.

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
Finally, the 2nd data structure is a ring list of players. A ring list is a structure with nodes which loop back around that way that after all the players are done in a turn, you can loop around and do things like remove the cards from the hand or keep looping around. There is still a semblance of who is the first and last player, but the player nodes never point out to nothing unless the player is being removed from the game.

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