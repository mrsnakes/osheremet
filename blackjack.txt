import random


class Deck:
    """
    Creating class object attributes, that will be used to build game deck
    """
    ranks = ("Two", "Three", "Four", "Five", "Six", "Seven", "Eight", "Nine", "Ten", "Jack", "Queen", "King", "Ace")
    suits = ('Hearts', 'Diamonds', 'Spades', 'Clubs')
    values = {'Two': 2, 'Three': 3, 'Four': 4, 'Five': 5, 'Six': 6, 'Seven': 7, 'Eight': 8, 'Nine': 9, 'Ten': 10,
              'Jack': 10, 'Queen': 10, 'King': 10, 'Ace': 11}
    deck = []

    def __init__(self):
        """
        Building & shuffling the deck, whenever class instance is created
        """
        self.deck = []  # Resetting deck, whenever class instance is created
        for rank in self.ranks:
            for suit in self.suits:
                self.deck.append((rank, suit))
        random.shuffle(self.deck)
        print("\n********************\nNEW GAME BEGINS NOW!\n")


class Hand:
    """
    Hand class to control each player cards, values and aces count
    """

    def __init__(self):
        self.hand = []  # Starting with an empty hand for each player(class instance)
        self.value = 0  # Starting with 0 value for each hand
        self.aces_count = 0  # Starting with 0 aces for each hand

    def get_card(self, deck):
        """
        Removing last(top) card from the deck and adding it to the hand
        :param deck: Game deck object
        :return: None
        """
        self.hand.append(deck.pop())

    def get_hand_value(self):
        """
        Calculating value of the hand, adjusting to possible Aces
        :return: Hand value
        """
        self.value = 0  # Resetting hand value, when method is called
        self.aces_count = 0  # Resetting aces count, when method is called
        for rank, suit in self.hand:
            self.value += Deck.values[rank]
        for rank, suite in self.hand:
            if rank == 'Ace':
                self.aces_count += 1
        if self.aces_count >= 1 and self.value > 21:
                self.value = self.value - 10 * self.aces_count
        return self.value

    def reveal_cards(self, player="Player", hide=False):
        """
        Revealing card(s) of the Player/Dealer
        :param player: str reference to Player's or Dealer's hand
        :param hide: bool; Used to hide Dealer's last card
        :return: None
        """
        if hide is False:
            print("{} cards: {}".format(player, self.hand))
        else:
            print("{} cards: [{}, ('????', '????')]".format(player, self.hand[0:-1]))


class Bets:
    """
    Bets class to organize Player's chips, bets and addition/subtrcation when he wins/loses
    """

    def __init__(self, total):
        """
        Added attributes to __init__ method to leave an option (for future) to add 2nd instance of Bets class. e.g.
        dealer's bets
        """
        self.total = total  # Asking player how much $$ he wants to play on
        self.bet = 0  # Starting with 0 bet
        self.betting = 1  # Flag to continue asking for a bet

    def win_bet(self):
        """
        Adds bet amount to player's total
        :return: None
        """
        self.total += self.bet

    def lose_bet(self):
        """
        Subtracts bet amount from player's total
        :return: None
        """
        self.total -= self.bet

    def take_bet(self):
        """
        Validates, whether player has enough $$ to bet
        Reserves bet amount from player's total
        :return: None
        """
        if self.total != 0:
            while self.betting:
                    self.bet = int(input("Enter your bet: \n"))
                    if self.bet == 0:
                        print("Bet should be greater than 0!")
                        self.take_bet()
                    elif self.bet > self.total:
                        print("Bet cannot exceed your total amount of {}".format(self.total))
                        self.take_bet()
                    else:
                        print("Player bet is : {}".format(self.bet))
                        self.betting = 0
                        break
        else:
            print("Not enough money to play.\n")
            restart_game(bets)


def hit_or_stand(deck, hand1, hand2, bets, limit):
    """
    Hit or Stand function, which determines whether Player/Dealer hits or stands
    :param deck: Game deck object
    :param hand1: Player's hand object
    :param hand2: Dealer's hand object
    :param bets: Player's bets object
    :param limit: Defines limit for each hand to stop hitting at, list of 2 integers
    :return: None
    """
    while hand1.get_hand_value() < int(limit[0]):
            h_or_s = str(input("Hit or Stand? H / S\n"))
            if h_or_s[0].lower() == "h":
                    print("Player hits >>> ")
                    hand1.get_card(deck.deck)
                    if busted(hand1, hand2, bets) is False and blackjack(hand1, hand2, bets) is False:
                        hand1.reveal_cards(player="Player", hide=False)
                        print("Player value: {}".format(hand1.get_hand_value()))
            elif h_or_s[0].lower() == "s":
                    print("Player stands...")
                    while hand2.get_hand_value() < int(limit[1]):
                        hand2.get_card(deck.deck)
                        if busted(hand1, hand2, bets) is False and blackjack(hand1, hand2, bets) is False:
                            print("\nDealer hits >>>")
                            hand2.reveal_cards(player="Dealer", hide=True)
                            continue
                    else:
                        define_winner(hand1, hand2, bets)
                        break


def drop_win(hand1, hand2, bets):
    """
    Function checks whether player or dealer got
    :param hand1: Player's hand object
    :param hand2: Dealer's hand object
    :param bets: Player's bets object
    :return: bool True, if player/dealer got blackjack (21)
    """
    if hand1.get_hand_value() == hand2.get_hand_value() == 21:
        print("BlackJack Tie on drop!")
        display_all_cards_and_values(hand1, hand2)
        continue_playing(bets)
        return True
    elif hand1.get_hand_value() == 21 and hand2.get_hand_value() < 21:
        print("Player BlackJack on drop!")
        display_all_cards_and_values(hand1, hand2)
        bets.win_bet()
        continue_playing(bets)
        return True
    elif hand2.get_hand_value() == 21 and hand1.get_hand_value() < 21:
        print("Dealer BlackJack on drop!")
        display_all_cards_and_values(hand1, hand2)
        bets.lose_bet()
        continue_playing(bets)
        return True
    else:
        return False


def busted(hand1, hand2, bets):
    """
    Function checks, whether player or dealer busted // got hand value over 21
    :param hand1: Player's hand object
    :param hand2: Dealer's hand object
    :param bets: Player's bets object
    :return: bool True if player/dealer busted
    """
    if hand1.get_hand_value() > 21:
        print("Player Busted :-|")
        display_all_cards_and_values(hand1, hand2)
        bets.lose_bet()
        continue_playing(bets)
        return True
    elif hand2.get_hand_value() > 21:
        print("Dealer Busted!")
        display_all_cards_and_values(hand1, hand2)
        bets.win_bet()
        continue_playing(bets)
        return True
    else:
        return False


def blackjack(hand1, hand2, bets):
    """
    Function checks, whether player or dealer got Blackjack (21)
    :param hand1: Player's hand object
    :param hand2: Dealer's hand object
    :param bets: Player's bets object
    :return: None
    """
    if hand1.get_hand_value() == 21 and hand2.get_hand_value() < 21:
        display_all_cards_and_values(hand1, hand2)
        print("Player BlackJack!")
        bets.win_bet()
        continue_playing(bets)
        return True
    elif hand2.get_hand_value() == 21 and hand1.get_hand_value() < 21:
        display_all_cards_and_values(hand1, hand2)
        print("Dealer BlackJack!")
        bets.lose_bet()
        continue_playing(bets)
        return True
    else:
        return False


def define_winner(hand1, hand2, bets):
    """
    Function defines winner, when both player and dealer stopped hitting and haven't busted or got blackjack
    :param hand1: Player's hand object
    :param hand2: Dealer's hand object
    :param bets: Player's bets object
    :return: None
    """
    print("\n********************\nDefining winner!\n")
    if hand1.get_hand_value() == hand2.get_hand_value():
        print("Tie!")
        display_all_cards_and_values(hand1, hand2)
        continue_playing(bets)
    elif hand1.get_hand_value() > hand2.get_hand_value():
        print("Player Wins!")
        display_all_cards_and_values(hand1, hand2)
        bets.win_bet()
        continue_playing(bets)
    else:
        print("Dealer Wins!")
        display_all_cards_and_values(hand1, hand2)
        bets.lose_bet()
        continue_playing(bets)


def continue_playing(bets):
    """
    Function asks player, whether he wants to continue playing, considering he has enough balance
    :param bets: Player's bets object
    :return: None
    """
    if bets.total > 0:
        answer = str(input("Your current balance is: {} . Want to continue playing? Y / N\n".format(bets.total)))
        if answer[0].lower() == 'y':
            pass
        elif answer[0].lower() == "n":
            global playing
            playing = 0
        else:
            print("Invalid Input")
            continue_playing(bets)
    else:
        print("Not enough money to play :(\n")
        restart_game(bets)


def restart_game(bets):
    """
    Function restarts the game from scratch, resetting total amount
    :param bets: Player's bets object
    :return: None
    """
    answer = str(input("Want to start a new game? Y / N\n"))
    if answer[0].lower() == 'y':
        bets.total = 100
        main()
    elif answer[0].lower() == "n":
        exit()
    else:
        print("Invalid Input")
        restart_game(bets)


def display_all_cards_and_values(hand1, hand2):
    """
    Function reveals all cards and prints each hand's value, when there's a winner
    :param hand1: Player's hand object
    :param hand2: Dealer's hand object
    :return: None
    """
    print("Player's cards : {}".format(hand1.hand))
    print("Player's value: {}".format(hand1.get_hand_value()))
    print("Dealer's cards: {}".format(hand2.hand))
    print("Dealer's value: {}".format(hand2.get_hand_value()))


def main():
    global playing
    playing = 1
    player_bets = Bets(total=100)
    while playing:
        game_deck = Deck()
        print("Player total is {}".format(player_bets.total))
        player_hand = Hand()
        dealer_hand = Hand()
        player_bets.take_bet()
        player_bets.betting = 1
        player_hand.get_card(game_deck.deck)
        player_hand.get_card(game_deck.deck)
        dealer_hand.get_card(game_deck.deck)
        dealer_hand.get_card(game_deck.deck)
        player_hand.reveal_cards(player="Player", hide=False)
        dealer_hand.reveal_cards(player="Dealer", hide=True)
        if drop_win(hand1=player_hand, hand2=dealer_hand, bets=player_bets) is False:
            print("Player value: {}".format(player_hand.get_hand_value()))
            hit_or_stand(deck=game_deck, hand1=player_hand, hand2=dealer_hand, bets=player_bets, limit=[21, 17])
        else:
            break


if __name__ == '__main__':
    main()
