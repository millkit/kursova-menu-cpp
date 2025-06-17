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

```cpp
#include <fstream>    
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
    } while (end != start && isspace(static_cast<unsigned char>(*end)));
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
```

```menu.ini
[Main]
1=File
2=Edit
3=View
4=Help
5=Tools

[File]
1=Create New...
2=Open
3=Save
4=Print
5=Back

[File->Create New...]
1=Text File .TXT
2=Excel Spreadsheet .XLSX
3=Word Document .DOCX
4=Image .PNG
5=Folder

[File->Create New...->Text File .TXT]
1=Blank
2=Template A
3=Template B
4=Encrypted
5=With Timestamp

[File->Create New...->Excel Spreadsheet .XLSX]
1=Budget
2=Report
3=Invoice
4=Calendar
5=Empty Sheet

[File->Create New...->Word Document .DOCX]
1=Letter
2=Resume
3=Report
4=Manual
5=Form

[File->Create New...->Image .PNG]
1=Blank Canvas
2=Logo Template
3=Photo Frame
4=Icon Set
5=Sketch Pad

[File->Create New...->Folder]
1=Project
2=Backup
3=Media
4=Archive
5=Temp


[Edit]
1=Undo
2=Redo
3=Cut
4=Copy
5=Paste

[Edit->Undo]
1=Last Action
2=Typing
3=Delete
4=Format Change
5=Move

[Edit->Redo]
1=Last Redone
2=Typing
3=Insert
4=Replace
5=Merge

[Edit->Cut]
1=Text
2=Image
3=Table
4=Section
5=Code Block

[Edit->Copy]
1=Text
2=Image
3=Chart
4=Style
5=Format

[Edit->Paste]
1=Plain Text
2=Keep Formatting
3=Merge Style
4=Paste as Link
5=Paste Special


[View]
1=Zoom
2=Layout
3=Toolbars
4=Fullscreen
5=Grid

[View->Zoom]
1=Zoom In
2=Zoom Out
3=Reset
4=Fit to Width
5=Custom

[View->Layout]
1=Print Layout
2=Web Layout
3=Draft View
4=Outline View
5=Read Mode

[View->Toolbars]
1=Standard
2=Formatting
3=Drawing
4=Review
5=Custom

[View->Fullscreen]
1=Enable
2=Disable
3=Presentation Mode
4=Immersive Mode
5=Split View

[View->Grid]
1=Show Grid
2=Hide Grid
3=Grid Spacing
4=Snap to Grid
5=Customize


[Help]
1=User Guide
2=FAQ
3=Shortcuts
4=Feedback
5=About

[Help->User Guide]
1=Getting Started
2=Editing Tools
3=Saving Files
4=Export Options
5=Advanced Features

[Help->FAQ]
1=General
2=Errors
3=Performance
4=Compatibility
5=Licensing

[Help->Shortcuts]
1=Editing
2=Navigation
3=View Control
4=File Management
5=Misc

[Help->Feedback]
1=Report Bug
2=Suggest Feature
3=Rate App
4=Write Review
5=Contact Support

[Help->About]
1=Version
2=License Info
3=Credits
4=Updates
5=Terms


[Tools]
1=Spelling
2=Translate
3=Macros
4=Extensions
5=Options

[Tools->Spelling]
1=Check Now
2=Auto Correct
3=Dictionary
4=Grammar
5=Language

[Tools->Translate]
1=Selected Text
2=Document
3=Phrasebook
4=Set Language
5=History

[Tools->Macros]
1=Run Macro
2=Record Macro
3=Edit Macro
4=Import
5=Delete

[Tools->Extensions]
1=Manage
2=Install
3=Update
4=Remove
5=Get More

[Tools->Options]
1=General
2=Display
3=Editing
4=Privacy
5=Backup

; Final action on selection example:
; When selecting: Tools -> Translate -> Document
; Console Output: "Path: Tools > Translate > Document"
```
