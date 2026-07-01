Online Supermarket Database
Module: Programming in C/C++ (IY469)
Name: Nabaa Adam
Student ID: P330417
Tutor Name: Aimen Djemaa 
GitHub: https://github.com/nabaadam24-ui/supermarket-database 

I confirm that this assignment is my own work. Where I have referred to academic or other sources, I have provided citations as comments in the code.




Item.h
#ifndef ITEM_H
#define ITEM_H
#include <string>
using namespace std;
// I confirm that this assignment is my own work. Where I have referred to
// academic or other sources, I have provided citations as comments in the code.
// Module: Programming in C/C++ (IY469)
// Name: Nabaa Adam
// StudentID: P330417
// i used a class here so i can group all the data for one product together
// and keep it private so it can only be changed through the get/set functions
class Item {
private:
    string name;
    string bestBefore;
    int quantity;
    double price;
    string type;

public:
    Item(string n, string bb, int q, double pr, string tp);

    string getName();
    string getBestBefore();
    int getQuantity();
    double getPrice();
    string getType();

    void setName(string n);
    void setQuantity(int q);
    void setPrice(double pr);
    void setType(string tp);
    void setBestBefore(string bb);

    string serialise();
    static Item deserialise(string line);
    void print();
};

#endif


Item.cpp
#include "Item.h"
#include <iostream>
#include <sstream>
#include <iomanip>
#include <stdexcept>
using namespace std;
// constructor checks all values are valid before saving them
// if anything is wrong it throws an exception
Item::Item(string n, string bb, int q, double pr, string tp) {
    if (n.empty())       throw invalid_argument("Name cannot be empty!");
    if (bb.size() != 10) throw invalid_argument("Date must be YYYY-MM-DD!");
    if (q < 0)           throw invalid_argument("Quantity cannot be negative!");
    if (pr < 0)          throw invalid_argument("Price cannot be negative!");
    if (tp.empty())      throw invalid_argument("Type cannot be empty!");
    name=n; bestBefore=bb; quantity=q; price=pr; type=tp;
}
string Item::getName()       { return name; }
string Item::getBestBefore() { return bestBefore; }
int    Item::getQuantity()   { return quantity; }
double Item::getPrice()      { return price; }
string Item::getType()       { return type; }
void Item::setName(string n)        { if (n.empty()) throw invalid_argument("Name empty!"); name=n; }
void Item::setQuantity(int q)       { if (q<0) throw invalid_argument("Quantity negative!"); quantity=q; }
void Item::setPrice(double pr)      { if (pr<0) throw invalid_argument("Price negative!"); price=pr; }
void Item::setType(string tp)       { if (tp.empty()) throw invalid_argument("Type empty!"); type=tp; }
void Item::setBestBefore(string bb) { if (bb.size()!=10) throw invalid_argument("Bad date!"); bestBefore=bb; }
// turns the item into one line of text so it can be saved to the file
string Item::serialise() {
    ostringstream out;
    out << fixed << setprecision(2) << price;
    return name+"|"+bestBefore+"|"+to_string(quantity)+"|"+out.str()+"|"+type;
}
// takes a line from the file and rebuilds the item
Item Item::deserialise(string line) {
    stringstream ss(line);
    string n,bb,qStr,prStr,tp;
    getline(ss,n,'|');
    getline(ss,bb,'|');
    getline(ss,qStr,'|');
    getline(ss,prStr,'|');
    getline(ss,tp);
    if (n.empty()) throw runtime_error("Corrupt data!");
    return Item(n,bb,stoi(qStr),stod(prStr),tp);
}
void Item::print() {
    cout << "Name       : " << name << endl;
    cout << "Best Before: " << bestBefore << endl;
    cout << "Quantity   : " << quantity << endl;
    cout << fixed << setprecision(2);
    cout << "Price      : £" << price << endl;
    cout << "Type       : " << type << endl;
    cout << "----------------------------" << endl;
}
Database.h
#ifndef DATABASE_H
#define DATABASE_H
#include "Item.h"
#include <vector>
#include <string>
using namespace std;
// i used a class here to group all the database operations together
// and store the items in a private vector so nothing can change it directly
class Database {
private:
    vector<Item> items;
    string filename = "supermarket.txt";
    int findIndex(string name);
public:
    bool isEmpty();
    void loadFromFile();
    void saveToFile();
    void listAll();
    void addItem(Item item);
    void deleteItem(string name);
    void editItem(string name, string newName, string newBB,
                  int newQty, double newPrice, string newType);
    void printLowStock();
    void printAboutToExpire();
    vector<Item> search(string nameFilter, string typeFilter);
    void saveResultsToFile(vector<Item>& results);
};

#endif

Database.cpp
#include "Database.h"
#include <iostream>
#include <fstream>
#include <algorithm>
#include <stdexcept>
using namespace std;
// uses find_if with a lambda to search the vector by name
// lambda and auto keyword from IY469 Week 5 Lecture Slides [3]
int Database::findIndex(string name) {
    auto match = [&name](Item& i) { return i.getName() == name; };
    auto it = find_if(items.begin(), items.end(), match);
    if (it == items.end()) return -1;
    return it - items.begin();
}
bool Database::isEmpty() { return items.empty(); }
// reads all items from the file when the program starts
void Database::loadFromFile() {
    ifstream file(filename);
    if (!file.is_open()) {
        cout << "[Info] No database file found. Starting fresh." << endl;
        return;
    }
    string line;
    while (getline(file, line)) {
        if (line.empty()) continue;
        try {
            items.push_back(Item::deserialise(line));
        } catch (exception& e) {
            // skip broken lines instead of crashing
            cout << "[Warning] Skipping bad line: " << e.what() << endl;
        }
    }
    file.close();
    cout << "[Info] Loaded " << items.size() << " item(s) from file." << endl;
}
// saves all items to the file after every change
void Database::saveToFile() {
    ofstream file(filename);
    if (!file.is_open()) throw runtime_error("Cannot open file for saving!");
    for (int i = 0; i < (int)items.size(); i++)
        file << items[i].serialise() << endl;
    file.close();
    cout << "[Info] Saved to file." << endl;
}
// milestone 1
void Database::listAll() {
    if (items.empty()) { cout << "Database is empty!" << endl; return; }
    cout << endl << "=== ALL ITEMS ===" << endl;
    for (int i = 0; i < (int)items.size(); i++) items[i].print();
}
// milestone 2
void Database::addItem(Item item) {
    if (findIndex(item.getName()) != -1)
        throw runtime_error("Item already exists: " + item.getName());
    items.push_back(item);
    saveToFile();
    cout << "[OK] Added: " << item.getName() << endl;
}
void Database::deleteItem(string name) {
    int idx = findIndex(name);
    if (idx == -1) throw runtime_error("Item not found: " + name);
    items.erase(items.begin() + idx);
    saveToFile();
    cout << "[OK] Deleted: " << name << endl;
}
void Database::editItem(string name, string newName, string newBB,
                         int newQty, double newPrice, string newType) {
    int idx = findIndex(name);
    if (idx == -1) throw runtime_error("Item not found: " + name);
    if (!newName.empty()) items[idx].setName(newName);
    if (!newBB.empty())   items[idx].setBestBefore(newBB);
    if (newQty >= 0)      items[idx].setQuantity(newQty);
    if (newPrice >= 0)    items[idx].setPrice(newPrice);
    if (!newType.empty()) items[idx].setType(newType);
    saveToFile();
    cout << "[OK] Updated: " << items[idx].getName() << endl;
}
// milestone 3
void Database::printLowStock() {
    cout << endl << "=== LOW STOCK (5 or less) ===" << endl;
    bool found = false;
    for (int i = 0; i < (int)items.size(); i++) {
        if (items[i].getQuantity() <= 5) { items[i].print(); found = true; }
    }
    if (!found) cout << "No low stock items." << endl;
}
void Database::printAboutToExpire() {
    cout << endl << "=== EXPIRING SOON (before 2026-07-01) ===" << endl;
    bool found = false;
    for (int i = 0; i < (int)items.size(); i++) {
        if (items[i].getBestBefore() < "2026-07-01") { items[i].print(); found = true; }
    }
    if (!found) cout << "No items expiring soon." << endl;
}
// milestone 4 - uses a lambda to filter results
// lambda syntax from IY469 Week 5 Lecture Slides [3]
vector<Item> Database::search(string nameFilter, string typeFilter) {
    vector<Item> results;
    auto matches = [&](Item& item) {
        bool nameOk = nameFilter.empty() || item.getName().find(nameFilter) != string::npos;
        bool typeOk = typeFilter.empty() || item.getType().find(typeFilter) != string::npos;
        return nameOk && typeOk;
    };
    for (int i = 0; i < (int)items.size(); i++)
        if (matches(items[i])) results.push_back(items[i]);
    return results;
}
// milestone 5
void Database::saveResultsToFile(vector<Item>& results) {
    if (results.empty()) throw runtime_error("No results to save!");
    ofstream file("results.txt");
    if (!file.is_open()) throw runtime_error("Cannot open results file!");
    file << "Name,Best Before,Quantity,Price,Type" << endl;
    for (int i = 0; i < (int)results.size(); i++)
        file << results[i].serialise() << endl;
    file.close();
    cout << "[OK] " << results.size() << " result(s) saved to results.txt!" << endl;
}


Main.cpp
#include "Database.h"
#include <iostream>
using namespace std;

// I confirm that this assignment is my own work. Where I have referred to
// academic or other sources, I have provided citations as comments in the code.
// Module: Programming in C/C++ (IY469)
// Name: Nabaa Adam
// StudentID: P330417

// References:
// [1] IY469 Week 3 Lecture Slides (2026) - Classes, encapsulation, private/public members
// [2] IY469 Week 4 Lecture Slides (2026) - Constructors and class member functions
// [3] IY469 Week 5 Lecture Slides (2026) - Lambda, auto, serialisation
// [4] IY469 C4 Week 1 Lecture Slides (2026) - Testing plans, error handling, exception handling
// [5] CodeBeauty (2020) 'C++ Programming' [YouTube channel]
//     Available at: https://www.youtube.com/@CodeBeauty
//     - Used for understanding classes, vectors and exception handling

string readString(string prompt) {
    cout << prompt;
    string val;
    getline(cin, val);
    return val;
}

int readInt(string prompt) {
    while (true) {
        cout << prompt;
        int val;
        if (cin >> val) { cin.ignore(1000, '\n'); return val; }
        cin.clear();
        cin.ignore(1000, '\n');
        cout << "[Error] Please enter a whole number." << endl;
    }
}

double readDouble(string prompt) {
    while (true) {
        cout << prompt;
        double val;
        if (cin >> val) { cin.ignore(1000, '\n'); return val; }
        cin.clear();
        cin.ignore(1000, '\n');
        cout << "[Error] Please enter a number like 1.99" << endl;
    }
}

void menuAdd(Database& db) {
    cout << endl << "-- Add New Item --" << endl;
    try {
        string name = readString("Name        : ");
        string bb   = readString("Best Before (YYYY-MM-DD): ");
        int    qty  = readInt("Quantity    : ");
        double pr   = readDouble("Price       : ");
        string type = readString("Type        : ");
        db.addItem(Item(name, bb, qty, pr, type));
    } catch (exception& e) {
        cout << "[Error] " << e.what() << endl;
    }
}

void menuDelete(Database& db) {
    cout << endl << "-- Delete Item --" << endl;
    try {
        string name = readString("Item name to delete: ");
        db.deleteItem(name);
    } catch (exception& e) {
        cout << "[Error] " << e.what() << endl;
    }
}

void menuEdit(Database& db) {
    cout << endl << "-- Edit Item --" << endl;
    cout << "(Press Enter to keep current value)" << endl;
    try {
        string name    = readString("Name of item to edit: ");
        string newName = readString("New name (or Enter)  : ");
        string newBB   = readString("New date (or Enter)  : ");
        string newQtyS = readString("New quantity (or -1) : ");
        string newPrS  = readString("New price (or -1)    : ");
        string newType = readString("New type (or Enter)  : ");
        int    qty   = newQtyS.empty() ? -1 : stoi(newQtyS);
        double price = newPrS.empty()  ? -1 : stod(newPrS);
        db.editItem(name, newName, newBB, qty, price, newType);
    } catch (exception& e) {
        cout << "[Error] " << e.what() << endl;
    }
}

void menuSearch(Database& db) {
    cout << endl << "-- Search --" << endl;
    cout << "(Leave blank to match everything)" << endl;
    string nameF = readString("Name filter: ");
    string typeF = readString("Type filter: ");
    vector<Item> results = db.search(nameF, typeF);
    if (results.empty()) { cout << "No items found." << endl; return; }
    cout << endl << "=== SEARCH RESULTS ===" << endl;
    for (int i = 0; i < (int)results.size(); i++) results[i].print();
    string save = readString("Save results to file? (y/n): ");
    if (save == "y" || save == "Y") {
        try { db.saveResultsToFile(results); }
        catch (exception& e) { cout << "[Error] " << e.what() << endl; }
    }
}

void showMenu() {
    cout << endl << "============================" << endl;
    cout << "   SUPERMARKET DATABASE" << endl;
    cout << "============================" << endl;
    cout << "1. List all items" << endl;
    cout << "2. Add item" << endl;
    cout << "3. Delete item" << endl;
    cout << "4. Edit item" << endl;
    cout << "5. Low stock / Expiring soon" << endl;
    cout << "6. Search for item" << endl;
    cout << "0. Exit" << endl;
    cout << "============================" << endl;
    cout << "Enter your choice: ";
}

void seedSampleData(Database& db) {
    struct Sample { const char* name; const char* date; int qty; double price; const char* cat; };
    Sample data[] = {
        {"Milk",          "2026-03-20", 25, 1.20, "Dairy"},
        {"Eggs",          "2026-03-18", 12, 2.10, "Dairy"},
        {"Bread",         "2026-03-15",  8, 1.00, "Bakery"},
        {"Apples",        "2026-04-01", 40, 0.30, "Produce"},
        {"Bananas",       "2026-03-16", 18, 0.25, "Produce"},
        {"Orange_Juice",  "2026-05-10", 30, 2.00, "Beverages"},
        {"Tomatoes",      "2026-03-17", 10, 0.45, "Produce"},
        {"Pasta",         "2027-01-01", 60, 0.80, "Pantry"},
        {"Rice",          "2027-02-10", 50, 1.10, "Pantry"},
        {"Yogurt",        "2026-03-19", 20, 0.90, "Dairy"},
        {"Butter",        "2026-06-01", 14, 1.80, "Dairy"},
        {"Cereal",        "2026-12-01", 22, 2.40, "Pantry"},
        {"Shampoo",       "2028-05-01", 35, 3.50, "Toiletries"},
        {"Chocolate_Bar", "2026-09-01", 45, 0.70, "Snacks"},
        {"Crisps",        "2026-07-15", 33, 0.90, "Snacks"}
    };
    for (int i = 0; i < 15; i++) {
        try { db.addItem(Item(data[i].name, data[i].date, data[i].qty, data[i].price, data[i].cat)); }
        catch (exception&) { }
    }
}

int main() {
    Database db;
    db.loadFromFile();

    if (db.isEmpty()) {
        cout << "[Info] Empty database - adding sample data." << endl;
        seedSampleData(db);
    }

    int choice = -1;
    while (choice != 0) {
        showMenu();
        cin >> choice;
        cin.ignore(1000, '\n');
        switch (choice) {
            case 1: db.listAll(); break;
            case 2: menuAdd(db); break;
            case 3: menuDelete(db); break;
            case 4: menuEdit(db); break;
            case 5: db.printLowStock(); db.printAboutToExpire(); break;
            case 6: menuSearch(db); break;
            case 0: cout << "Goodbye!" << endl; break;
            default: cout << "Invalid choice." << endl;
        }
    }
    return 0;
}
