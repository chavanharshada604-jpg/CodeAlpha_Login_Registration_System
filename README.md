// CodeAlpha C++ Internship - Task 2: Login and Registration System
// NOTE: Passwords are salted + hashed (educational). For real apps use bcrypt/argon2.

#include <iostream>
#include <fstream>
#include <string>
#include <sstream>
#include <functional>
#include <cctype>
#include <cstdlib>
#include <ctime>
#include <limits>
using namespace std;

const string USER_FILE = "users.txt"; // format: username salt hash

// ---------- Helper functions ----------

// Discard the rest of the current input line (handles extra spaces/text)
void clearLine() {
    cin.ignore(numeric_limits<streamsize>::max(), '\n');
}

string makeSalt() {
    const string chars = "abcdefghijklmnopqrstuvwxyz0123456789";
    string s;
    for (int i = 0; i < 8; i++) s += chars[rand() % chars.size()];
    return s;
}

string hashPassword(const string &password, const string &salt) {
    size_t h = hash<string>{}(salt + password);
    stringstream ss;
    ss << h;
    return ss.str();
}

bool validUsername(const string &u) {
    if (u.length() < 3 || u.length() > 15) return false;
    for (char c : u)
        if (!isalnum((unsigned char)c) && c != '_') return false;
    return true;
}

bool validPassword(const string &p) {
    if (p.length() < 6) return false;
    bool digit = false, alpha = false;
    for (char c : p) {
        if (isdigit((unsigned char)c)) digit = true;
        if (isalpha((unsigned char)c)) alpha = true;
    }
    return digit && alpha;
}

bool usernameExists(const string &username) {
    ifstream in(USER_FILE);
    string u, salt, h;
    while (in >> u >> salt >> h)
        if (u == username) return true;
    return false;
}

// ---------- Registration ----------
void registerUser() {
    string username, password;
    cout << "\n--- REGISTRATION ---\n";

    cout << "Username (3-15 letters/digits/_): ";
    if (!(cin >> username)) return;
    clearLine();
    if (!validUsername(username)) {
        cout << "Error: Invalid username format.\n";
        return;
    }
    if (usernameExists(username)) {
        cout << "Error: Username already taken. Try another.\n";
        return;
    }

    cout << "Password (min 6 chars, letters + digits): ";
    if (!(cin >> password)) return;
    clearLine();
    if (!validPassword(password)) {
        cout << "Error: Weak password. Use at least 6 characters with letters and digits.\n";
        return;
    }

    string salt = makeSalt();
    ofstream out(USER_FILE, ios::app);
    if (!out) {
        cout << "Error: Could not open file.\n";
        return;
    }
    out << username << " " << salt << " " << hashPassword(password, salt) << "\n";
    cout << "Success: Registration completed! You can now login.\n";
}

// ---------- Login ----------
void loginUser() {
    string username, password;
    cout << "\n--- LOGIN ---\n";

    cout << "Username: ";
    if (!(cin >> username)) return;
    clearLine();
    cout << "Password: ";
    if (!(cin >> password)) return;
    clearLine();

    ifstream in(USER_FILE);
    if (!in) {
        cout << "Error: No users registered yet.\n";
        return;
    }

    string u, salt, h;
    while (in >> u >> salt >> h) {
        if (u == username) {
            if (hashPassword(password, salt) == h)
                cout << "Success: Login successful! Welcome, " << username << ".\n";
            else
                cout << "Error: Incorrect password.\n";
            return;
        }
    }
    cout << "Error: Username not found.\n";
}

// ---------- Main ----------
int main() {
    srand((unsigned)time(0));
    int choice;

    do {
        cout << "\n=========================\n";
        cout << "  LOGIN & REGISTRATION\n";
        cout << "=========================\n";
        cout << "1. Register\n2. Login\n3. Exit\nChoice: ";

        if (!(cin >> choice)) {
            if (cin.eof()) break;   // input ended, stop the program
            cin.clear();
            clearLine();
            choice = 0;
        } else {
            clearLine();
        }

        switch (choice) {
            case 1: registerUser(); break;
            case 2: loginUser(); break;
            case 3: cout << "Goodbye!\n"; break;
            default: cout << "Invalid choice. Try again.\n";
        }
    } while (choice != 3);

    return 0;
}
