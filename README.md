include <iostream>
#include <fstream>
#include <sstream>
#include <string>
#include <vector>
#include <algorithm>
#include <stdexcept>
using namespace std;

// ============================================================
// I confirm that this assignment is my own work. Where I have
// referred to academic or other sources, I have provided
// citations as comments in the code.
// Module: Programming in C/C++ (IY469)
// Assessment: Online Supermarket Database
// Name: Nabaa Adam
// StudentID: P330417
// ============================================================

class Item {
private:
    string name;
    string bestBefore;
    int    quantity;
    double price;
    string type;

public:
    Item(string n, string bb, int q, double pr, string tp) {
        if (n.empty())       throw invalid_argument("Name cannot be empty!");
        if (bb.size() != 10) throw invalid_argument("Date must be YYYY-MM-DD!");
        if (q < 0)           throw invalid_argument("Quantity cannot be negative!");
        if (pr < 0)          throw invalid_argument("Price cannot be negative!");
        if (tp.empty())      throw invalid_argument("Type cannot be empty!");
        name=n; bestBefore=bb; quantity=q; price=pr; type=tp;
    }

    string getName()       { return name; }
    string getBestBefore() { return bestBefore; }
    int    getQuantity()   { return quantity; }
    double getPrice()      { return price; }
    string getType()       { return type; }

    void setName(string n)       { if (n.empty()) throw invalid_argument("Name empty!"); name=n; }
    void setQuantity(int q)      { if (q<0) throw invalid_argument("Quantity negative!"); quantity=q; }
    void setPrice(double pr)     { if (pr<0) throw invalid_argument("Price negative!"); price=pr; }
    void setType(string tp)      { if (tp.empty()) throw invalid_argument("Type empty!"); type=tp; }
    void setBestBefore(string bb){ if (bb.size()!=10) throw invalid_argument("Bad date!"); bestBefore=bb; }

    // SERIALISE - turn item into one line of text for the file
    string serialise() {
        return name+"|"+bestBefore+"|"+to_string(quantity)+"|"+to_string(price)+"|"+type;
    }

    // DESERIALISE - turn one line from the file back into an Item
    static Item deserialise(string line) {
        stringstream ss(line);
        string n,bb,qStr,pStr,tp;
        getline(ss,n,'|');
        getline(ss,bb,'|');
        getline(ss,qStr,'|');
        getline(ss,pStr,'|');
        getline(ss,tp,'|');
        if (n.empty()) throw runtime_error("Corrupt data!");
        return Item(n,bb,stoi(qStr),stod(pStr),tp);
    }

    void print() {
        cout << "Name       : " << name       << endl;
        cout << "Best Before: " << bestBefore << endl;
        cout << "Quantity   : " << quantity   << endl;
        cout << "Price      : £" << price     << endl;
        cout << "Type       : " << type       << endl;
        cout << "----------------------------" << endl;
    }
};

class Database {
private:
    vector<Item> items;
    string filename = "supermarket.txt"; // THE FILE

    int findIndex(string name) {
        auto match = [&name](Item& i) { return i.getName() == name; };
        auto it = find_if(items.begin(), items.end(), match);
        if (it == items.end()) return -1;
        return it - items.begin();
    }

public:
    // LOAD - read all items from the file when program starts
    void loadFromFile() {
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
                cout << "[Warning] Skipping bad line: " << e.what() << endl;
            }
        }
        file.close();
        cout << "[Info] Loaded " << items.size() << " item(s) from file." << endl;
    }

    // SAVE - write all items to the file after every change
    void saveToFile() {
        ofstream file(filename);
        if (!file.is_open())
            throw runtime_error("Cannot open file for saving!");
        for (Item& item : items)
            file << item.serialise() << "\n";
        file.close();
        cout << "[Info] Saved to file." << endl;
    }

    // MILESTONE 1 - List all items
    void listAll() {
        if (items.empty()) { cout << "Database is empty!" << endl; return; }
        cout << "\n=== ALL ITEMS ===" << endl;
        for (Item& item : items) item.print();
    }

    // MILESTONE 2 - Add item
    void addItem(Item item) {
        if (findIndex(item.getName()) != -1)
            throw runtime_error("Item already exists: " + item.getName());
        items.push_back(item);
        saveToFile(); // save after every change
        cout << "[OK] Added: " << item.getName() << endl;
    }

    // MILESTONE 2 - Delete item
    void deleteItem(string name) {
        int idx = findIndex(name);
        if (idx == -1) throw runtime_error("Item not found: " + name);
        items.erase(items.begin() + idx);
        saveToFile(); // save after every change
        cout << "[OK] Deleted: " << name << endl;
    }

    // MILESTONE 2 - Edit item
    void editItem(string name, string newName, string newBB,
                  int newQty, double newPrice, string newType) {
        int idx = findIndex(name);
        if (idx == -1) throw runtime_error("Item not found: " + name);
        Item& item = items[idx];
        if (!newName.empty())  item.setName(newName);
        if (!newBB.empty())    item.setBestBefore(newBB);
        if (newQty >= 0)       item.setQuantity(newQty);
        if (newPrice >= 0)     item.setPrice(newPrice);
        if (!newType.empty())  item.setType(newType);
        saveToFile(); // save after every change
        cout << "[OK] Updated: " << item.getName() << endl;
    }

    // MILESTONE 3 - Low stock
    void printLowStock() {
        cout << "\n=== LOW STOCK (5 or less) ===" << endl;
        bool found = false;
        // LAMBDA - checks if item quantity is low
        auto isLow = [](Item& item) { return item.getQuantity() <= 5; };
        for (Item& item : items) {
            if (isLow(item)) { item.print(); found = true; }
        }
        if (!found) cout << "No low stock items." << endl;
    }

    // MILESTONE 3 - About to expire
    void printAboutToExpire() {
        cout << "\n=== EXPIRING SOON (before 2026-07-01) ===" << endl;
        bool found = false;
        // LAMBDA - checks if item expires soon
        auto expiringSoon = [](Item& item) {
            return item.getBestBefore() < "2026-07-01";
        };
        for (Item& item : items) {
            if (expiringSoon(item)) { item.print(); found = true; }
        }
        if (!found) cout << "No items expiring soon." << endl;
    }

    // MILESTONE 4 - Search
    vector<Item> search(string nameFilter, string typeFilter) {
        vector<Item> results;
        // LAMBDA - checks if item matches the search filters
        auto matches = [&](Item& item) {
            bool nameOk = nameFilter.empty() ||
                item.getName().find(nameFilter) != string::npos;
            bool typeOk = typeFilter.empty() ||
                item.getType().find(typeFilter) != string::npos;
            return nameOk && typeOk;
        };
        for (Item& item : items) {
            if (matches(item)) results.push_back(item);
        }
        return results;
    }

    // MILESTONE 5 - Save results to a FILE
    void saveResultsToFile(vector<Item>& results) {
        if (results.empty())
            throw runtime_error("No results to save!");

        string outFile = "results.txt";
        ofstream file(outFile);
        if (!file.is_open())
            throw runtime_error("Cannot open results file!");

        // Write to the file
        file << "Name,Best Before,Quantity,Price,Type\n";
        for (Item& item : results) {
            file << item.serialise() << "\n";
        }
        file.close();

        cout << "[OK] " << results.size()
             << " result(s) saved to results.txt!" << endl;
    }
};

// ── INPUT HELPERS ─────────────────────────────────────────────
// Safe string input
string readString(string prompt) {
    cout << prompt;
    string val;
    cin.ignore();
    getline(cin, val);
    return val;
}

// Safe integer input
int readInt(string prompt) {
    while (true) {
        cout << prompt;
        int val;
        if (cin >> val) return val;
        cin.clear();
        cin.ignore(1000, '\n');
        cout << "[Error] Please enter a whole number." << endl;
    }
}

// Safe double input
double readDouble(string prompt) {
    while (true) {
        cout << prompt;
        double val;
        if (cin >> val) return val;
        cin.clear();
        cin.ignore(1000, '\n');
        cout << "[Error] Please enter a number like 1.99" << endl;
    }
}

// ── MENU FUNCTIONS ────────────────────────────────────────────
void menuAdd(Database& db) {
    cout << "\n-- Add New Item --" << endl;
    try {
        string name = readString("Name        : ");
        string bb   = readString("Best Before (YYYY-MM-DD): ");
        int    qty  = readInt   ("Quantity    : ");
        double pr   = readDouble("Price       : ");
        string type = readString("Type        : ");
        db.addItem(Item(name, bb, qty, pr, type));
    } catch (exception& e) {
        cout << "[Error] " << e.what() << endl;
    }
}

void menuDelete(Database& db) {
    cout << "\n-- Delete Item --" << endl;
    try {
        string name = readString("Item name to delete: ");
        db.deleteItem(name);
    } catch (exception& e) {
        cout << "[Error] " << e.what() << endl;
    }
}

void menuEdit(Database& db) {
    cout << "\n-- Edit Item --" << endl;
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
    cout << "\n-- Search --" << endl;
    cout << "(Leave blank to match everything)" << endl;
    string nameF = readString("Name filter: ");
    string typeF = readString("Type filter: ");
    vector<Item> results = db.search(nameF, typeF);
    if (results.empty()) {
        cout << "No items found." << endl;
        return;
    }
    cout << "\n=== SEARCH RESULTS ===" << endl;
    for (Item& item : results) item.print();

    // MILESTONE 5 - offer to save results to file
    string save = readString("Save results to file? (y/n): ");
    if (save == "y" || save == "Y") {
        try {
            db.saveResultsToFile(results);
        } catch (exception& e) {
            cout << "[Error] " << e.what() << endl;
        }
    }
}

void showMenu() {
    cout << "\n============================" << endl;
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

// ── MAIN ──────────────────────────────────────────────────────
int main() {
    Database db;

    // Load items from file when program starts
    db.loadFromFile();

    int choice = -1;
    while (choice != 0) {
        showMenu();
        cin >> choice;

        switch (choice) {
            case 1: db.listAll();             break;
            case 2: menuAdd(db);              break;
            case 3: menuDelete(db);           break;
            case 4: menuEdit(db);             break;
            case 5:
                db.printLowStock();
                db.printAboutToExpire();
                break;
            case 6: menuSearch(db);           break;
            case 0: cout << "Goodbye!" << endl; break;
            default: cout << "Invalid choice." << endl;
        }
    }

    return 0;
}
