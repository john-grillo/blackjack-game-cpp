// Blackjack.cpp : Defines the entry point for the console application.
// Authors: Kurtis Pittman, Gabe Wahrmund, John Grillo
// CS161 Programming Project 
// Sources: Cplusplus website (srand function and time.h library, system("cls") function)
// This is a simulation of a single player versus the computer game of blackjack. Payout is equal to the 
// initial bet for a win, equal to the initial bet time 1.5 for a score of 21 and no payout for a tie.
//

#include "stdafx.h"
#include <iostream>
#include <string>
#include <iomanip>
#include <ctime>

using namespace std;

//Global Variables
int score;
char restart;
double walletAdjusted = 100;// set the player wallet
double dealerAdjusted = 10000;// set the bank
double playerBet;
int dlrValue = 0 , plrValue = 0;
int deal = 0;
bool soft_17 = false;
int cur_plr_card = 0;// tracks the array location of the last card dealt to the player hand. Begins with array location [0]
int cur_dlr_card = 0;// tracks the array location of the last card dealt to the dealer hand. Begins with array location [0]

//structs
struct deck_of_cards
{
	string card;
	string suite;
	int value;
};



// Global struct arrays
deck_of_cards player_hand[11];
deck_of_cards dealer_hand[11];
deck_of_cards set_deck[52];
deck_of_cards shuffled_deck[52];
deck_of_cards temp_deck[52];

// functions
void shuffle_deck();// shuffles the deck of cards for active play
void setting_deck();// sets up a deck of cards in an array
void playHand ();// play Blackjack!
double adjust (int dh, int ph, double bet);// function to adjust wallet and bank
double wallet (double adjust);// function holds player wallet amount
double dealer (double adjust);// function holds bank amount
void eval_Player_Aces(int curPlace);
void eval_Dealer_Aces(int curPlace);
void player_value_calc();
void dealer_value_calc();
void dealer_soft17(int curPlace);
void clear_hand_out();
void deal_player_card();
void deal_dealer_card();
void show_player_hand();
void show_dealer_hand();
void play_game();
void results(int ph, int dh);


int _tmain()
{
	bool menu = true;
	char option;

	cout << fixed << showpoint << setprecision(2);
	
	cout << "Welcome to the table! We'll be playing blackjack!" << endl << endl;
	
	do
	{
		cout << "The menu options are as follows:  " << endl;
		cout << "'Q' or 'q' will quit the program  " << endl;
		cout << "'B' or 'b' will show the bank.  " << endl;
		cout << "'P' or 'p' will deal the cards and begin play  " << endl;
		cout << "'W' or 'w' will bring your wallet to the screen.  " << endl;
		cout << "All other keys will repeat the menu options.  " << endl;
		cin >> option;
		cout << endl;

		//the switch structure to make the menu possible.
		switch (option)
		{
			case 'B':
			case 'b':
				cout << endl;
				cout << "Bank: $" << dealer(0) << endl;					
				cout << endl;
				system("pause");
				system("cls");
				break;

			case 'P':
			case 'p':				
				system("cls");

				cout << endl;
				cout << "Setting up deck......" << endl << endl;
				setting_deck();

				system("pause");
				system("cls");
	
				cout << endl;
				cout << "Shuffling Deck......." << endl << endl;
				shuffle_deck();		

				system("pause");
				system("cls");	

				play_game();	

				if (walletAdjusted == 0)
					option = 'q';

			case 'W':
			case 'w':
				cout << endl;
				cout << "Wallet: $" << walletAdjusted << endl;
				cout << endl;
				system("pause");
				system("cls");
				break;

			case 'Q':
			case 'q':	
				option = 'q';
				cout << "You got to know when to hold 'em, know when to fold 'em, Know when to walk away and know when to run.  " << endl;
				cout << "Your wallet contains: $" << walletAdjusted << endl;
				if (walletAdjusted == 0)
					walletAdjusted = 100;

				if (dealerAdjusted == 0)
					dealerAdjusted = 10000;
				return 0;				
		}
	}//end of do while loop & update condition.	
	while (option != 'q');
	
	return 0;
}
void shuffle_deck()
{
	for (int x = 0; x < 52; x++)// prepares the deck for shuffling ***********************************
	{
		shuffled_deck[x].card = set_deck[x].card;
		shuffled_deck[x].suite = set_deck[x].suite;
		shuffled_deck[x].value = set_deck[x].value;

		temp_deck[x].card = set_deck[x].card;// temp_deck and shuffled_deck are the same at this point
		temp_deck[x].suite = set_deck[x].suite;
		temp_deck[x].value = set_deck[x].value;
	}//************************************************************************************************

	srand(unsigned int (time(NULL)));
	for (int shuffle = 1; shuffle < (5001 + (rand() % 20000)); shuffle++)// shuffles the deck *******************************
	{
		int hi = 9; // offset for upper half of deck array
		int lo = 0; // offset for lower half of deck array

		for (int y = 0; y < 18; y++)// shuffle 1st 1/3 of deck ***************************************************************
		{
			shuffled_deck[y].card = temp_deck[hi].card;
			shuffled_deck[y].suite = temp_deck[hi].suite;
			shuffled_deck[y].value = temp_deck[hi].value;

			y++;

			shuffled_deck[y].card = temp_deck[lo].card;
			shuffled_deck[y].suite = temp_deck[lo].suite;
			shuffled_deck[y].value = temp_deck[lo].value;

			hi++;
			lo++;
		}// *************************** end shuffle 1st 1/3 of deck ***********************************************************

		hi = 26; // offset for upper half of deck array
		lo = 18; // offset for lower half of deck array

		for (int y = 18; y < 34; y++)// shuffle 2nd 1/3 of deck ***************************************************************

		{
			shuffled_deck[y].card = temp_deck[hi].card;
			shuffled_deck[y].suite = temp_deck[hi].suite;
			shuffled_deck[y].value = temp_deck[hi].value;

			y++;

			shuffled_deck[y].card = temp_deck[lo].card;
			shuffled_deck[y].suite = temp_deck[lo].suite;
			shuffled_deck[y].value = temp_deck[lo].value;

			hi++;
			lo++;
		}// **************************** end shuffle 2nd 1/3 of deck ***********************************************************

		hi = 43; // offset for upper half of deck array
		lo = 34; // offset for lower half of deck array

		for (int y = 34; y < 52; y++)// shuffle 3rd 1/3 of deck ***************************************************************
		{
			shuffled_deck[y].card = temp_deck[hi].card;
			shuffled_deck[y].suite = temp_deck[hi].suite;
			shuffled_deck[y].value = temp_deck[hi].value;

			y++;

			shuffled_deck[y].card = temp_deck[lo].card;
			shuffled_deck[y].suite = temp_deck[lo].suite;
			shuffled_deck[y].value = temp_deck[lo].value;

			hi++;
			lo++;
		}// **************************** end shuffle 3rd 1/3 of deck ***********************************************************

		for (int x = 0; x < 52; x++)// resets temp deck for next iteration of the shuffle
		{
			temp_deck[x].card = shuffled_deck[x].card;
			temp_deck[x].suite = shuffled_deck[x].suite;
			temp_deck[x].value = shuffled_deck[x].value;
		}





	}//***********************************************************************************************************************


}
void setting_deck()
{
	int counter = 0;
	int z = 0;
	string crd_suite;
	
	for (int x = 1; x < 5; x++)
	{
		counter++;
			
		switch (x)
		{
		case 1:
			crd_suite = " of Spades";
			break;
		case 2:
			crd_suite = " of Clubs";
			break;
		case 3:
			crd_suite = " of Diamonds";
			break;
		case 4:
			crd_suite = " of Hearts";
			break;
			
		}		
				
		for (int y = 0; y < 13; y++)
		{
			if (y == 0)
			{
				set_deck[y + z].card = "Ace";
				set_deck[y + z].suite = crd_suite;
				set_deck[y + z].value = 11;
				

			}
			else if (y == 9)
			{
				set_deck[y + z].card = "10";
				set_deck[y + z].suite = crd_suite;
				set_deck[y + z].value = 10;
				
			}
			else if (y == 10)
			{
				set_deck[y + z].card = "Jack";
				set_deck[y + z].suite = crd_suite;
				set_deck[y + z].value = 10;
				
			}
			else if (y == 11)
			{
				set_deck[y + z].card = "Queen";
				set_deck[y + z].suite = crd_suite;
				set_deck[y + z].value = 10;
				
			}
			else if (y == 12)
			{
				set_deck[y + z].card = "King";
				set_deck[y + z].suite = crd_suite;
				set_deck[y + z].value = 10;
				
			}
			else //if (y > 1)
			{
				set_deck[y + z].card = static_cast<char>(y + 49);
				set_deck[y + z].suite = crd_suite;
				set_deck[y + z].value = y + 1;
				
			}			
		}

		z = z + 13;
	}
	
}
void playHand()
{
	char action;

	if (deal == 48)
	{		
		shuffle_deck();
		deal = 0;
	}	
		
	for (int x = 0; x < 2; x++)// deal out 2 cards per player (4 cards from shuffled deck)
	{
		deal_dealer_card();
		
		// check for aces in the dealers hand
		if (dlrValue > 21)
			eval_Dealer_Aces(cur_dlr_card);		
		
		deal_player_card();

		// check for aces in the players hand
		if (plrValue > 21)
			eval_Player_Aces(cur_plr_card);
			
	}// end of initial deal ******************************************************************************	
			
	cout << endl << endl;// Show the hand in play for player perspective ***********************************************
	cout << ">>Dealers Hand<<" << endl;
	cout << "Dealer hole card" <<  endl;
	cout << dealer_hand[1].card << dealer_hand[1].suite << endl;	
	
	show_player_hand();
	
	cout << endl << endl;
	if (plrValue > 20)	
		return;	

	cout << "{S}tay or {H}it? ";
	cin >> action;
	cout << endl;

	if ((action == 'h') || (action == 'H'))// **************************** begin player hand in play section ***************************
	{		
		do
		{
			system("cls");

			deal_player_card();

			// check for aces in the players hand
			if (plrValue > 21)
				eval_Player_Aces(cur_plr_card);			

			system("cls");

			cout << endl << endl;// Show the hand in play ***********************************************
			cout << ">>Dealers Hand<<" << endl;
			cout << "Dealer hole card" <<  endl;
			cout << dealer_hand[1].card << dealer_hand[1].suite << endl;
			// default to show dealer function after player stays****************************************

			show_player_hand();
	
			cout << endl << endl;
			if (plrValue > 20)
				return;

			if (plrValue == 21)
			{				
				return;
			}

			cout << "{S}tay or {H}it? ";
			cin >> action;
			cout << endl;
		}
		while ((action == 'h') || (action == 'H'));
	}// ******************************************************* end of player hand in play section ***************************

	//#########################################################################################################################
	//#########################################################################################################################

	system("cls");

	cout << endl << endl;// Show the hand in play including dealer hand ***********************************************
		
	show_player_hand();
	
	show_dealer_hand();	

	cout << endl << endl;

	if (dlrValue > 20)	
		action = 's';
	
	if (dlrValue > 16)
		eval_Dealer_Aces(cur_dlr_card);	

	if (dlrValue < 17)
		action = 'h';
	else 
		action = 's';

	cout << " Press a key to continue dealers hand......";
	system("pause");
	
		

	if ((action == 'h') || (action == 'H'))// **************************** begin dealer hand in play section ***************************
	{		
		do
		{
			system("cls");

			deal_dealer_card();

			if (dlrValue == 21)
				return;
			
			
			// check for aces in the dealers hand
			if (dlrValue > 16)
				eval_Dealer_Aces(cur_dlr_card);			

			system("cls");

			cout << endl << endl;// Show the hand in play ***********************************************
			show_player_hand();

			show_dealer_hand();	
	
			cout << endl << endl;
			if (dlrValue > 20)	
				action = 's';
	
			if (dlrValue > 16)
				eval_Dealer_Aces(cur_dlr_card);	

			if (dlrValue < 17)
				action = 'h';
			else 
				action = 's';
			
			cout << " Press a key to continue dealers hand......";
			system("pause");
		}
		while ((action == 'h') || (action == 'H'));
	}// ******************************************************* end of dealer hand in play section ***************************	

	
}
double wallet (double adjust) // function holds player bank
{
	walletAdjusted = walletAdjusted + adjust;
	return walletAdjusted;
}
double dealer (double adjust) // function holds dealer bank
{
	dealerAdjusted = dealerAdjusted - adjust;
	return dealerAdjusted;
}
double adjust (int dh, int ph, double bet) // function returns amount to adjust banks
{
	double outcome;

	if (dh == ph)
	{
		outcome = 0; // returns push
		return outcome;
	}
	else if	(ph == 21)
	{
		outcome = bet * 1.5; // returns blackjack
		return outcome;
	}
	else if (ph > 21)
	{
		outcome = -bet; // returns loss
		return outcome;
	}
	else if (dh > 21)
	{
		outcome = bet; // returns win
		return outcome;
	}
	else if (dh < ph)
	{
		outcome = bet; // returns win
		return outcome;
	}
	else if (dh > ph)
	{
		outcome = -bet; // returns loss
		return outcome;
	}
	
	
	outcome = 0;
	return outcome;
}
void eval_Player_Aces(int curPlace)
{// this will assume that the hand value evaluates to over 21 already		
	
	for (int x = 0; x <= curPlace; x++)
	{
		if (player_hand[x].card == "Ace")
		{
			player_hand[x].value = 1;
			player_value_calc();
			if (plrValue < 22)
				return;
		}
	}


}
void eval_Dealer_Aces(int curPlace)
{// this will assume that the hand value evaluates to over 16 already
	
	for (int x = 0; x <= curPlace; x++)
	{		
		if (dealer_hand[x].card == "Ace")
		{
			dealer_hand[x].value = 1;
			dealer_value_calc();
			
			if (dlrValue < 17)
				return;

		}
	}
}
void player_value_calc()
{
	plrValue = 0;
	for (int x = 0; x < 11; x++)
		plrValue = plrValue + player_hand[x].value;
}
void dealer_value_calc()
{
	dlrValue = 0;
	for (int x = 0; x < 11; x++)
		dlrValue = dlrValue + dealer_hand[x].value;
}
void dealer_soft17(int curPlace)
{
	soft_17 = false;

	if (dlrValue == 21)
		return;

	for (int x = 0; x <= curPlace; x++)
	{		
		if (dealer_hand[x].card == "Ace")
		{
			soft_17 = true;
			return;
		}
	}
}
void clear_hand_out()
{
	for (int x = 0; x < 12; x++)// clear player and dealer hands out ************************************
	{
		dealer_hand[x].card = "";
		dealer_hand[x].suite = "";
		dealer_hand[x].value = 0;			
		dlrValue = 0;

		player_hand[x].card = "";
		player_hand[x].suite = "";
		player_hand[x].value = 0;			
		plrValue = 0;
	}
}
void deal_player_card()
{
	player_hand[cur_plr_card].card = shuffled_deck[deal].card;
	player_hand[cur_plr_card].suite = shuffled_deck[deal].suite;
	player_hand[cur_plr_card].value = shuffled_deck[deal].value;			
	player_value_calc();
	cur_plr_card++;
	deal++;
	
	if (deal == 48)
	{		
		shuffle_deck();
		deal = 0;
	}
}
void deal_dealer_card()
{
	dealer_hand[cur_dlr_card].card = shuffled_deck[deal].card;
	dealer_hand[cur_dlr_card].suite = shuffled_deck[deal].suite;
	dealer_hand[cur_dlr_card].value = shuffled_deck[deal].value;			
	dealer_value_calc();
	deal++;
	cur_dlr_card++;
	
	if (deal == 48)
	{		
		shuffle_deck();
		deal = 0;
	}
}
void show_player_hand()
{
	cout << endl;	
	cout << "Current Bet: $" << playerBet << endl << endl;
	cout << ">>Players Hand<<" << endl;
	for (int x = 0; x < cur_plr_card; x++)
	{
		cout << player_hand[x].card << player_hand[x].suite << endl;
	}

	cout << endl << endl;	
	cout << "You hand currently comes to: " << plrValue << endl;
}
void show_dealer_hand()
{
	cout << endl << endl;
	cout << ">>Dealer's Hand<<" << endl;
	for (int x = 0; x < cur_dlr_card; x++)
	{
		cout << dealer_hand[x].card << dealer_hand[x].suite << endl;
	}

	cout << endl << endl;	
	cout << "Dealer's hand currently comes to: " << dlrValue << endl;
}
void play_game()
{
	while (true)
	{	
		if (walletAdjusted == 0) // game over
		{
			cout << "Game Over." << endl;
			cout << "Restart? (Y/N): ";
			cin >> restart;
			cout << endl;
			system("cls");

			switch (restart)
			{
				case 'y':
				case 'Y':
				{
					walletAdjusted = 100;
					dealerAdjusted = 10000;
					break;
				}
				default:
					{
						system("cls");
						return;
					}
			}
		}
		cout << endl;
		cout << "Wallet: $" << wallet(0) << endl;		
		cout << endl;

		cout << "Enter your bet for this hand (-99 to quit play): "; // inputs for bank functions
		cin >> playerBet;

		if (playerBet == -99)
			return;

		while (cin.fail())
		{			
			cin.clear();
			cin.ignore(1, '/n');
			playerBet = 0;
			cout << "Enter your bet for this hand: "; // inputs for bank functions
			cin >> playerBet;
		}

			


		cout << endl;

		

		if (walletAdjusted - playerBet < 0) // overbet
		{
			cout << "Sorry, you do not have enough money to make that bet." << endl;
			cout << endl;
		}
		else if (playerBet < .01)
			cout << "Illegal Bet. Try again.....) " << endl;		
		else // play!
		{
			system("cls");
			
			playHand();

			results(plrValue, dlrValue);

			wallet (adjust (dlrValue, plrValue, playerBet));
			dealer (adjust (dlrValue, plrValue, playerBet));

			system("pause");
			
			cout << endl;			
			system("cls");
			
			// reset for next hand
			plrValue = 0;
			dlrValue = 0;
			clear_hand_out();
			cur_plr_card = 0;
			cur_dlr_card = 0;
			soft_17 = false;
			playerBet = 0;


		}
	}
}
void results(int ph, int dh)
{
	if (dh == ph)
	{
		cout << endl;
		cout << "Push. No Winner. " << endl;
		cout << endl;
	}
	else if (ph > 21)
	{
		cout << endl;
		cout << "This hand is a bust! " << endl;
		cout << endl;
	}
	else if (dh > 21)
	{
		cout << endl;
		cout << "Dealer busts! You win! " << endl;
		cout << endl;
	}

	else if (dh < ph)
	{
		cout << endl;
		cout << "You Win! " << endl;
		cout << endl;
	}

	else if (dh > ph)
	{
		cout << endl;
		cout << "Dealer Wins! " << endl;
		cout << endl;
	}

	else if	(ph == 21)
	{
		cout << endl;
		cout << "21! You Win! " << endl;
		cout << endl;
	}

}
