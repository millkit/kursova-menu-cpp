# Консольний додаток з багаторівневим меню

Програма на C++, що зчитує меню з INI-файлу та дозволяє користувачеві взаємодіяти з багаторівневими пунктами через консоль.

## Файли:
- main.cpp — основний код
- menu.ini — структура меню
- Kursova_robota.exe — виконуваний файл для Windows

## Технології:
- C++
- STL: map, vector, pair, string
- Робота з файлами та структура INI

- ## Код:
- #include <fstream>    
#include <map>        
#include <vector>     
#include <string>     
#include <sstream>    
#include <algorithm>  
#include <cctype>     

using namespace std;  

// Оголошення типів для зручності та читабельності коду
using MenuItem = pair<string, string>;               
using MenuSection = vector<MenuItem>;                
using MenuLevel = map<string, MenuSection>;          

// Допоміжна функція, яка видаляє пробіли з початку і кінця вхідного рядка
static inline string trim(const string& s) {  
    auto start = s.begin();  
    while (start != s.end() && isspace(static_cast<unsigned char>(*start))) ++start;  
    if (start == s.end()) 
return "";  
    auto end = s.end();  
    do { 
--end;
 	} while (end != start && isspace(static_cast<unsigned char>(*end)));  // 
    return string(start, end + 1);  
}

// Функція завантаження меню з текстового INI-файлу
MenuLevel loadMenu(const string& filename) {
    ifstream file(filename);  
    if (!file) {  
        cerr << "File is not found: " << filename << endl;
        exit(1);  
    }
    MenuLevel menu;  // Основна структура меню (словник секцій)
    string line, section;  
    while (getline(file, line)) {  
        line = trim(line);  
        if (line.empty() || line[0] == ';') continue;  
        if (line.front() == '[' && line.back() == ']') {  
            section = line.substr(1, line.size() - 2);  
        }
        else if (!section.empty()) {  
            size_t eq = line.find('=');  
            if (eq != string::npos) {
                string key = trim(line.substr(0, eq));  
                string value = trim(line.substr(eq + 1));  
                menu[section].push_back({ key, value });  
            }
        }
    }
    return menu;  
}

// Функція для виведення на екран пунктів меню поточної секції та зчитування вибору користувача
int showMenu(const MenuSection& items, const string& prompt) {
    cout << "\n" << prompt << endl;  
    for (const auto& [key, value] : items) {  
        cout << key << ". " << value << endl;  
    }
    cout << "0. Exit\n> ";  
    int choice;
    cin >> choice;  
    return choice;  
}

// Основна функція програми, де виконується логіка навігації по меню
int main() {
    MenuLevel menu = loadMenu("menu.ini");  
    vector<string> path;  
    string section = "Main";  
    
    while (true) {  
        auto it = menu.find(section);  
        if (it == menu.end() || it->second.empty()) {  
            cout << "\nYour choice: ";
            for (size_t i = 0; i < path.size(); ++i) {  
                if (i) cout << " -> ";
                cout << path[i];
            }
            cout << endl;
            cout << "0. Back to main menu\n> ";  
            int back;
            cin >> back;
            if (back == 0) {
                section = "Main";  
                path.clear();  
            }
            continue;  
        }

        int choice = showMenu(it->second, (path.empty() ? "Main menu:" : "Menu: " + section));  
        if (choice == 0) {  
            if (path.empty()) {
                break;  
                path.pop_back();  
            }
            if (path.empty()) {
                section = "Main";
            }
            else if (path.size() == 1) {
                section = path[0];
            }
            else {
                section = path[0];  
                for (size_t i = 1; i < path.size(); ++i)
                    section += "->" + path[i];
            }
            continue;  
        }

        auto& items = it->second;  // Отримуємо список пунктів меню
        auto found = find_if(items.begin(), items.end(), [choice](const MenuItem& p) {
            return stoi(p.first) == choice;  
            });

        if (found == items.end()) {  
            cout << "Incorrect choice!\n";
            continue;  
        }

        path.push_back(found->second);  
        section = path[0];  
        for (size_t i = 1; i < path.size(); ++i)
            section += "->" + path[i];

           system("cls");  
    }

    cout << "Exit.\n";  
    cin.get();  
    system("pause");  
    return 0;  
}

