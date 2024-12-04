#include <iostream>
#include <vector>
#include <string>
#include <fstream>
#include <iomanip>

using namespace std;

class CreditCard {
private:
    string cardHolderName;
    string cardNumber;
    string expiryDate;
    string cardType;
    vector<string> transactionHistory;
    float profitPercent;

public:
    CreditCard(string holderName, string cardNum, string expDate, string type)
        : cardHolderName(holderName), cardNumber(cardNum), expiryDate(expDate), cardType(type) {
        if (type == "Golden")
            profitPercent = 20;
        else if (type == "Silver")
            profitPercent = 10;
        else if (type == "Bronze")
            profitPercent = 5;
    }

    string getCardHolderName() const { return cardHolderName; }
    string getCardNumber() const { return cardNumber; }
    string getCardType() const { return cardType; }
    float getProfitPercent() const { return profitPercent; }

    void addTransaction(const string& transaction, float amount) {
        float profit = (amount * profitPercent) / 100;
        string fullTransaction = transaction + " (Amount: Rs. " + to_string(amount) +
                                 ", Profit: Rs. " + to_string(profit) + ")";
        transactionHistory.push_back(fullTransaction);
    }

    void viewCardDetails(ofstream& outFile) const {
        string border = "||==============================================    ||";
        outFile << border << " \n";
        outFile << "||               CREDIT CARD DETAILS                ||\n";
        outFile << border << "\n";
        outFile << "|| Card Holder: " << setw(36) << left << cardHolderName << "||\n";
        outFile << "|| Card Number: " << setw(36) << left << cardNumber << "||\n";
        outFile << "|| Expiry Date: " << setw(36) << left << expiryDate << "||\n";
        outFile << "|| Card Type:   " << setw(36) << left << cardType << "||\n";
        outFile << border << " \n";
        outFile << "||             TRANSACTION HISTORY                  ||\n";
        if (transactionHistory.empty()) {
            outFile << "|| No transactions available.                       ||\n";
        } else {
            for (size_t i = 0; i < transactionHistory.size(); ++i) {
                string transaction = transactionHistory[i];
                if (transaction.size() > 36) {
                    transaction = transaction.substr(0, 33) + "...";
                }
                outFile << "|| " << setw(48) << left << transaction << "||\n";
            }
        }
        outFile << border << "\n\n";
    }
};

class User {
private:
    string username;
    string password;
    vector<CreditCard> cards;

public:
    void registerUser() {
        cout << "Enter Username: ";
        cin >> username;
        cout << "Enter Password: ";
        cin >> password;
        cout << "Registration Successful!\n";
    }

    bool loginUser() {
        string inputUsername, inputPassword;
        cout << "Enter Username: ";
        cin >> inputUsername;
        cout << "Enter Password: ";
        cin >> inputPassword;
        if (inputUsername == username && inputPassword == password) {
            cout << "Login Successful!\n";
            return true;
        }
        cout << "Invalid Credentials. Please try again.\n";
        return false;
    }

    void addCard() {
        string name, cardNum, expDate, type;
        cout << "Enter Card Holder's Name: ";
        cin.ignore();
        getline(cin, name);
        cout << "Enter Card Number: ";
        getline(cin, cardNum);
        cout << "Enter Expiry Date (MM/YY): ";
        getline(cin, expDate);
        cout << "Enter Card Type (Golden/Silver/Bronze): ";
        getline(cin, type);

        cards.push_back(CreditCard(name, cardNum, expDate, type));
        cout << "Card added successfully!\n";
    }

    void viewCards() const {
        if (cards.empty()) {
            cout << "No cards available.\n";
            return;
        }
        for (size_t i = 0; i < cards.size(); ++i) {
            cout << i + 1 << ". " << cards[i].getCardHolderName()
                 << "'s Card (Number: " << cards[i].getCardNumber() << ")\n";
        }
    }

    void addTransactionToCard() {
        size_t index;
        string description;
        float amount;

        viewCards();
        cout << "Enter the card index: ";
        cin >> index;
        if (index < 1 || index > cards.size()) {
            cout << "Invalid card index.\n";
            return;
        }

        cin.ignore();
        cout << "Enter Transaction Description: ";
        getline(cin, description);
        cout << "Enter Transaction Amount: ";
        cin >> amount;

        cards[index - 1].addTransaction(description, amount);
        cout << "Transaction added successfully!\n";
    }

    void saveToFile() const {
        ofstream outFile("CardDetails.txt");
        if (!outFile.is_open()) {
            cout << "Error opening file for writing.\n";
            return;
        }

        for (const auto& card : cards) {
            card.viewCardDetails(outFile);
        }
        cout << "All card details saved to 'CardDetails.txt'.\n";
        outFile.close();
    }

    void adminMode() const {
        cout << "\n--- Admin Mode ---\n";
        if (cards.empty()) {
            cout << "No cards available.\n";
            return;
        }
        for (size_t i = 0; i < cards.size(); ++i) {
            const CreditCard& card = cards[i];
            cout << "\nCard " << i + 1 << ": \n";
            cout << "Holder Name: " << card.getCardHolderName() << "\n";
            cout << "Card Number: " << card.getCardNumber() << "\n";
            cout << "Card Type: " << card.getCardType() << "\n";
            cout << "Profit Percentage: " << card.getProfitPercent() << "%\n";
        }
    }
};

int main() {
    User user;
    int choice;

    cout << "Welcome to Credit Card Management System!\n";
    cout << "1. Register\n2. Login\n3. Admin Mode\nEnter your choice: ";
    cin >> choice;

    if (choice == 1) {
        user.registerUser();
    } else if (choice == 2) {
        if (!user.loginUser()) {
            return 0;
        }
    } else if (choice == 3) {
        cout << "Enter Admin Mode (Password: admin123): ";
        string adminPass;
        cin >> adminPass;
        if (adminPass == "admin123") {
            user.adminMode();
            return 0;
        } else {
            cout << "Invalid Admin Password!\n";
            return 0;
        }
    }

    while (true) {
        cout << "\n--- Menu ---\n";
        cout << "1. Add a new card\n";
        cout << "2. View all cards\n";
        cout << "3. Add a transaction to a card\n";
        cout << "4. Save card details to file\n";
        cout << "5. Exit\n";
        cout << "Enter your choice: ";
        cin >> choice;

        if (choice == 1) {
            user.addCard();
        } else if (choice == 2) {
            user.viewCards();
        } else if (choice == 3) {
            user.addTransactionToCard();
        } else if (choice == 4) {
            user.saveToFile();
        } else if (choice == 5) {
            cout << "Thank you for using the system. Goodbye!\n";
            break;
        } else {
            cout << "Invalid choice! Please try again.\n";
        }
    }

    return 0;
}
