+++
date = '2025-12-16T23:59:50+08:00'
draft = false
title = '[C++] 檔案處理：從 C-style 到現代 C++'
categories = ['C++ Notes']
tags = ['C++']
+++

# 目錄

- [目錄](#目錄)
  - [前言](#前言)
  - [C++ 檔案處理的演進](#c-檔案處理的演進)
  - [方法一：C-style 檔案處理（FILE\*）](#方法一c-style-檔案處理file)
    - [基本操作](#基本操作)
    - [檔案開啟模式](#檔案開啟模式)
    - [優點與缺點](#優點與缺點)
  - [方法二：C++ 串流處理（fstream）](#方法二c-串流處理fstream)
    - [基本使用](#基本使用)
    - [三種檔案串流類別](#三種檔案串流類別)
    - [開啟模式](#開啟模式)
    - [完整的讀寫範例](#完整的讀寫範例)
  - [二進位檔案處理](#二進位檔案處理)
    - [寫入和讀取結構](#寫入和讀取結構)
    - [輸出結果](#輸出結果)
  - [錯誤處理](#錯誤處理)
    - [檢查檔案狀態](#檢查檔案狀態)
    - [完整的錯誤處理範例](#完整的錯誤處理範例)
  - [檔案位置操作](#檔案位置操作)
    - [seekg 和 seekp](#seekg-和-seekp)
    - [取得檔案大小](#取得檔案大小)
  - [C++17 filesystem 函式庫](#c17-filesystem-函式庫)
    - [基本操作](#基本操作-1)
    - [路徑處理](#路徑處理)
    - [檔案資訊查詢](#檔案資訊查詢)
  - [實用範例](#實用範例)
    - [範例 1：CSV 檔案處理](#範例-1csv-檔案處理)
    - [範例 2：設定檔處理](#範例-2設定檔處理)
    - [範例 3：檔案備份工具](#範例-3檔案備份工具)
  - [最佳實踐](#最佳實踐)
    - [DO（應該做的）](#do應該做的)
    - [DON'T（不應該做的）](#dont不應該做的)
  - [小結](#小結)

## 前言

檔案處理是程式設計中的基本操作。C++ 提供了多種檔案處理方式，從 C 語言繼承的 FILE* API，到 C++ 的串流（Stream）API，再到 C++17 引入的 filesystem 函式庫。

本文將介紹這些不同的檔案處理方式，並說明它們的適用場景和最佳實踐。

## C++ 檔案處理的演進

| 版本 | 方式 | 特色 |
|------|------|------|
| C | FILE* | 低階、靈活、需手動管理 |
| C++98 | fstream | 物件導向、RAII、型別安全 |
| C++17 | filesystem | 跨平台、路徑處理、檔案操作 |

## 方法一：C-style 檔案處理（FILE*）

### 基本操作

```cpp
#include <cstdio>
#include <iostream>

void cStyleExample() {
    // 1. 開啟檔案
    FILE* file = fopen("data.txt", "w");

    if (!file) {
        std::cerr << "無法開啟檔案\n";
        return;
    }

    // 2. 寫入資料
    fprintf(file, "Hello, World!\n");
    fprintf(file, "Number: %d\n", 42);

    // 3. 關閉檔案（必須手動）
    fclose(file);

    // 讀取檔案
    file = fopen("data.txt", "r");
    if (file) {
        char buffer[256];
        while (fgets(buffer, sizeof(buffer), file)) {
            std::cout << buffer;
        }
        fclose(file);
    }
}
```

### 檔案開啟模式

```cpp
// 文字模式
"r"   // 讀取（檔案必須存在）
"w"   // 寫入（建立新檔或覆蓋舊檔）
"a"   // 附加（寫入到檔案末端）
"r+"  // 讀寫（檔案必須存在）
"w+"  // 讀寫（建立新檔或覆蓋舊檔）
"a+"  // 讀取和附加

// 二進位模式（加 'b'）
"rb"  // 二進位讀取
"wb"  // 二進位寫入
"ab"  // 二進位附加
```

### 優點與缺點

**優點：**
- 效能較好（直接呼叫系統 API）
- 適合處理大型檔案
- 支援更底層的操作

**缺點：**
- 需要手動管理資源（容易忘記 fclose）
- 不是型別安全的
- 錯誤處理較繁瑣
- 不符合 RAII 原則

## 方法二：C++ 串流處理（fstream）

### 基本使用

```cpp
#include <fstream>
#include <iostream>
#include <string>

void cppStreamExample() {
    // 1. 寫入檔案（RAII 自動管理）
    {
        std::ofstream outFile("data.txt");

        if (!outFile) {
            std::cerr << "無法開啟檔案\n";
            return;
        }

        outFile << "Hello, World!\n";
        outFile << "Number: " << 42 << "\n";

    }  // 離開作用域時自動關閉

    // 2. 讀取檔案
    {
        std::ifstream inFile("data.txt");

        if (!inFile) {
            std::cerr << "無法開啟檔案\n";
            return;
        }

        std::string line;
        while (std::getline(inFile, line)) {
            std::cout << line << "\n";
        }
    }  // 自動關閉
}
```

### 三種檔案串流類別

```cpp
#include <fstream>

// 1. ifstream - 輸入檔案串流（讀取）
std::ifstream inFile("input.txt");

// 2. ofstream - 輸出檔案串流（寫入）
std::ofstream outFile("output.txt");

// 3. fstream - 雙向檔案串流（讀寫）
std::fstream file("data.txt", std::ios::in | std::ios::out);
```

### 開啟模式

```cpp
#include <fstream>
#include <iostream>

void openModes() {
    // ios::in - 讀取模式
    std::ifstream in("file.txt", std::ios::in);

    // ios::out - 寫入模式（預設會清空檔案）
    std::ofstream out("file.txt", std::ios::out);

    // ios::app - 附加模式（寫入到末端）
    std::ofstream append("file.txt", std::ios::app);

    // ios::trunc - 截斷模式（清空檔案內容）
    std::ofstream trunc("file.txt", std::ios::trunc);

    // ios::binary - 二進位模式
    std::ofstream binary("file.bin", std::ios::binary);

    // 組合模式
    std::fstream file("data.txt", std::ios::in | std::ios::out | std::ios::app);
}
```

### 完整的讀寫範例

```cpp
#include <fstream>
#include <iostream>
#include <string>

// 寫入文字檔
void writeTextFile(const std::string& filename) {
    std::ofstream file(filename);

    if (!file.is_open()) {
        std::cerr << "無法開啟檔案: " << filename << "\n";
        return;
    }

    file << "姓名,年齡,城市\n";
    file << "Alice,25,Taipei\n";
    file << "Bob,30,Kaohsiung\n";
    file << "Charlie,35,Taichung\n";

    std::cout << "檔案寫入完成: " << filename << "\n";
}

// 讀取文字檔（逐行）
void readTextFileLineByLine(const std::string& filename) {
    std::ifstream file(filename);

    if (!file.is_open()) {
        std::cerr << "無法開啟檔案: " << filename << "\n";
        return;
    }

    std::string line;
    int lineNumber = 0;

    while (std::getline(file, line)) {
        std::cout << "Line " << ++lineNumber << ": " << line << "\n";
    }
}

// 讀取文字檔（逐字）
void readTextFileWordByWord(const std::string& filename) {
    std::ifstream file(filename);

    if (!file.is_open()) {
        std::cerr << "無法開啟檔案: " << filename << "\n";
        return;
    }

    std::string word;

    while (file >> word) {  // 以空白字元分隔
        std::cout << word << "\n";
    }
}

// 讀取整個檔案內容
void readEntireFile(const std::string& filename) {
    std::ifstream file(filename);

    if (!file.is_open()) {
        std::cerr << "無法開啟檔案: " << filename << "\n";
        return;
    }

    // 方法 1：使用 string stream
    std::stringstream buffer;
    buffer << file.rdbuf();
    std::string content = buffer.str();

    std::cout << "檔案內容:\n" << content << "\n";

    // 方法 2：使用 iterator（C++11）
    file.seekg(0);  // 回到檔案開頭
    std::string content2(
        (std::istreambuf_iterator<char>(file)),
        std::istreambuf_iterator<char>()
    );
}

int main() {
    writeTextFile("data.csv");
    std::cout << "\n--- 逐行讀取 ---\n";
    readTextFileLineByLine("data.csv");

    std::cout << "\n--- 逐字讀取 ---\n";
    readTextFileWordByWord("data.csv");

    return 0;
}
```

## 二進位檔案處理

### 寫入和讀取結構

```cpp
#include <fstream>
#include <iostream>
#include <vector>
#include <cstring>

struct Student {
    char name[50];
    int age;
    double score;
};

// 寫入二進位檔案
void writeBinaryFile(const std::string& filename) {
    std::ofstream file(filename, std::ios::binary);

    if (!file) {
        std::cerr << "無法開啟檔案\n";
        return;
    }

    // 建立學生資料
    std::vector<Student> students = {
        {"Alice", 20, 95.5},
        {"Bob", 22, 88.0},
        {"Charlie", 21, 92.3}
    };

    // 複製字串到固定大小的陣列
    for (auto& s : students) {
        Student temp = s;
        strncpy(temp.name, s.name, sizeof(temp.name) - 1);
        temp.name[sizeof(temp.name) - 1] = '\0';

        file.write(reinterpret_cast<const char*>(&temp), sizeof(Student));
    }

    std::cout << "寫入 " << students.size() << " 筆資料\n";
}

// 讀取二進位檔案
void readBinaryFile(const std::string& filename) {
    std::ifstream file(filename, std::ios::binary);

    if (!file) {
        std::cerr << "無法開啟檔案\n";
        return;
    }

    Student student;

    std::cout << "讀取的學生資料:\n";
    while (file.read(reinterpret_cast<char*>(&student), sizeof(Student))) {
        std::cout << "姓名: " << student.name
                  << ", 年齡: " << student.age
                  << ", 分數: " << student.score << "\n";
    }
}

int main() {
    writeBinaryFile("students.dat");
    std::cout << "\n";
    readBinaryFile("students.dat");

    return 0;
}
```

### 輸出結果

```
寫入 3 筆資料

讀取的學生資料:
姓名: Alice, 年齡: 20, 分數: 95.5
姓名: Bob, 年齡: 22, 分數: 88
姓名: Charlie, 年齡: 21, 分數: 92.3
```

## 錯誤處理

### 檢查檔案狀態

```cpp
#include <fstream>
#include <iostream>

void checkFileState() {
    std::ifstream file("nonexistent.txt");

    // 方法 1：使用 is_open()
    if (!file.is_open()) {
        std::cerr << "檔案無法開啟\n";
    }

    // 方法 2：使用 operator bool()
    if (!file) {
        std::cerr << "檔案無法開啟\n";
    }

    // 方法 3：檢查特定狀態
    if (file.fail()) {
        std::cerr << "讀寫操作失敗\n";
    }

    if (file.bad()) {
        std::cerr << "嚴重錯誤（資料損毀）\n";
    }

    if (file.eof()) {
        std::cout << "到達檔案末端\n";
    }

    // 清除錯誤狀態
    file.clear();
}
```

### 完整的錯誤處理範例

```cpp
#include <fstream>
#include <iostream>
#include <string>

void safeFileOperation(const std::string& filename) {
    std::ifstream file(filename);

    if (!file.is_open()) {
        std::cerr << "錯誤：無法開啟檔案 " << filename << "\n";
        return;
    }

    std::string line;
    int lineNumber = 0;

    while (std::getline(file, line)) {
        lineNumber++;

        if (file.bad()) {
            std::cerr << "錯誤：讀取檔案時發生嚴重錯誤\n";
            break;
        }

        std::cout << lineNumber << ": " << line << "\n";
    }

    if (file.eof()) {
        std::cout << "成功讀取完整個檔案\n";
    } else if (file.fail()) {
        std::cerr << "警告：讀取檔案時發生錯誤\n";
    }
}
```

## 檔案位置操作

### seekg 和 seekp

```cpp
#include <fstream>
#include <iostream>

void filePositionOperations() {
    // 建立測試檔案
    {
        std::ofstream out("test.txt");
        out << "0123456789ABCDEF";
    }

    std::fstream file("test.txt", std::ios::in | std::ios::out);

    if (!file) {
        std::cerr << "無法開啟檔案\n";
        return;
    }

    // tellg() - 取得讀取位置
    std::cout << "初始讀取位置: " << file.tellg() << "\n";

    // seekg() - 設定讀取位置
    file.seekg(5);  // 移到第 5 個位元組
    char ch;
    file.get(ch);
    std::cout << "位置 5 的字元: " << ch << "\n";  // '5'

    // 從末端往回移動
    file.seekg(-3, std::ios::end);
    file.get(ch);
    std::cout << "倒數第 3 個字元: " << ch << "\n";  // 'D'

    // 從目前位置移動
    file.seekg(-1, std::ios::cur);
    file.get(ch);
    std::cout << "目前位置的字元: " << ch << "\n";  // 'D'

    // tellp() - 取得寫入位置
    // seekp() - 設定寫入位置（用法同 seekg）
    std::cout << "目前寫入位置: " << file.tellp() << "\n";
}
```

### 取得檔案大小

```cpp
#include <fstream>
#include <iostream>

size_t getFileSize(const std::string& filename) {
    std::ifstream file(filename, std::ios::binary | std::ios::ate);

    if (!file) {
        return 0;
    }

    // ios::ate 會將位置設定在檔案末端
    return file.tellg();
}

int main() {
    size_t size = getFileSize("test.txt");
    std::cout << "檔案大小: " << size << " bytes\n";

    return 0;
}
```

## 方法三：C++17 filesystem 函式庫

C++17 引入的 `<filesystem>` 標準函式庫提供了現代化、跨平台的檔案系統操作功能，包括：

- 檔案和目錄的建立、刪除、重新命名
- 路徑處理和操作
- 檔案資訊查詢（大小、權限、修改時間等）
- 目錄遍歷和內容列舉
- 複製、移動檔案和目錄

### 簡單範例

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

int main() {
    // 檢查檔案是否存在
    if (fs::exists("test.txt")) {
        // 取得檔案大小
        auto size = fs::file_size("test.txt");
        std::cout << "檔案大小: " << size << " bytes\n";

        // 複製檔案
        fs::copy_file("test.txt", "test_copy.txt");
    }

    return 0;
}
```

### 更多資訊

關於 C++17 filesystem 的完整說明，包括目錄操作、路徑處理、檔案遍歷等進階功能，請參考：

**[[C++] 目錄處理：C++17 filesystem 完整指南](./Cpp-DirectoryHandling.md)**

該文件涵蓋：
- 目錄的建立、刪除、遍歷
- 路徑操作和組合
- 檔案系統資訊查詢
- 實用範例（備份工具、檔案搜尋、清理工具等）
- 跨平台考量和最佳實踐

## 實用範例

### 範例 1：CSV 檔案處理

```cpp
#include <fstream>
#include <iostream>
#include <sstream>
#include <vector>
#include <string>

struct Person {
    std::string name;
    int age;
    std::string city;
};

// 寫入 CSV
void writeCSV(const std::string& filename, const std::vector<Person>& people) {
    std::ofstream file(filename);

    if (!file) {
        std::cerr << "無法開啟檔案\n";
        return;
    }

    // 寫入標題
    file << "Name,Age,City\n";

    // 寫入資料
    for (const auto& p : people) {
        file << p.name << "," << p.age << "," << p.city << "\n";
    }

    std::cout << "CSV 檔案寫入完成\n";
}

// 讀取 CSV
std::vector<Person> readCSV(const std::string& filename) {
    std::vector<Person> people;
    std::ifstream file(filename);

    if (!file) {
        std::cerr << "無法開啟檔案\n";
        return people;
    }

    std::string line;

    // 跳過標題行
    std::getline(file, line);

    // 讀取資料
    while (std::getline(file, line)) {
        std::stringstream ss(line);
        std::string name, city, ageStr;

        std::getline(ss, name, ',');
        std::getline(ss, ageStr, ',');
        std::getline(ss, city, ',');

        Person p{name, std::stoi(ageStr), city};
        people.push_back(p);
    }

    return people;
}

int main() {
    // 寫入
    std::vector<Person> people = {
        {"Alice", 25, "Taipei"},
        {"Bob", 30, "Kaohsiung"},
        {"Charlie", 35, "Taichung"}
    };

    writeCSV("people.csv", people);

    // 讀取
    auto readPeople = readCSV("people.csv");

    std::cout << "\n讀取的資料:\n";
    for (const auto& p : readPeople) {
        std::cout << p.name << " (" << p.age << ") - " << p.city << "\n";
    }

    return 0;
}
```

### 範例 2：設定檔處理

```cpp
#include <fstream>
#include <iostream>
#include <map>
#include <sstream>
#include <string>

class ConfigFile {
private:
    std::map<std::string, std::string> data;
    std::string filename;

public:
    explicit ConfigFile(const std::string& fname) : filename(fname) {}

    // 載入設定檔
    bool load() {
        std::ifstream file(filename);

        if (!file) {
            return false;
        }

        std::string line;
        while (std::getline(file, line)) {
            // 跳過空行和註解
            if (line.empty() || line[0] == '#' || line[0] == ';') {
                continue;
            }

            // 解析 key=value
            auto pos = line.find('=');
            if (pos != std::string::npos) {
                std::string key = line.substr(0, pos);
                std::string value = line.substr(pos + 1);

                // 移除空白
                key.erase(0, key.find_first_not_of(" \t"));
                key.erase(key.find_last_not_of(" \t") + 1);
                value.erase(0, value.find_first_not_of(" \t"));
                value.erase(value.find_last_not_of(" \t") + 1);

                data[key] = value;
            }
        }

        return true;
    }

    // 儲存設定檔
    bool save() const {
        std::ofstream file(filename);

        if (!file) {
            return false;
        }

        for (const auto& [key, value] : data) {
            file << key << " = " << value << "\n";
        }

        return true;
    }

    // 取得值
    std::string get(const std::string& key, const std::string& defaultValue = "") const {
        auto it = data.find(key);
        return (it != data.end()) ? it->second : defaultValue;
    }

    // 設定值
    void set(const std::string& key, const std::string& value) {
        data[key] = value;
    }

    // 顯示所有設定
    void print() const {
        for (const auto& [key, value] : data) {
            std::cout << key << " = " << value << "\n";
        }
    }
};

int main() {
    ConfigFile config("settings.ini");

    // 設定一些值
    config.set("username", "alice");
    config.set("port", "8080");
    config.set("debug", "true");

    // 儲存
    if (config.save()) {
        std::cout << "設定檔儲存成功\n";
    }

    // 載入
    ConfigFile loadedConfig("settings.ini");
    if (loadedConfig.load()) {
        std::cout << "\n載入的設定:\n";
        loadedConfig.print();

        std::cout << "\n取得特定值:\n";
        std::cout << "Username: " << loadedConfig.get("username") << "\n";
        std::cout << "Port: " << loadedConfig.get("port") << "\n";
    }

    return 0;
}
```

### 範例 3：檔案備份工具

```cpp
#include <filesystem>
#include <fstream>
#include <iostream>
#include <chrono>
#include <iomanip>
#include <sstream>

namespace fs = std::filesystem;

class FileBackup {
public:
    // 建立備份
    static bool backup(const std::string& sourceFile) {
        if (!fs::exists(sourceFile)) {
            std::cerr << "來源檔案不存在: " << sourceFile << "\n";
            return false;
        }

        // 產生備份檔名（加上時間戳記）
        std::string backupFile = generateBackupName(sourceFile);

        try {
            fs::copy_file(sourceFile, backupFile,
                         fs::copy_options::overwrite_existing);
            std::cout << "備份完成: " << backupFile << "\n";
            return true;
        } catch (const fs::filesystem_error& e) {
            std::cerr << "備份失敗: " << e.what() << "\n";
            return false;
        }
    }

private:
    static std::string generateBackupName(const std::string& filename) {
        fs::path p(filename);
        std::string stem = p.stem().string();
        std::string ext = p.extension().string();

        // 取得當前時間
        auto now = std::chrono::system_clock::now();
        auto time_t = std::chrono::system_clock::to_time_t(now);

        std::stringstream ss;
        ss << stem << "_backup_"
           << std::put_time(std::localtime(&time_t), "%Y%m%d_%H%M%S")
           << ext;

        return (p.parent_path() / ss.str()).string();
    }
};

int main() {
    // 建立測試檔案
    {
        std::ofstream file("important_data.txt");
        file << "This is important data!\n";
    }

    // 建立備份
    FileBackup::backup("important_data.txt");

    return 0;
}
```

## 最佳實踐

### DO（應該做的）

1. **使用 RAII 自動管理檔案**
   ```cpp
   // 好：自動關閉
   {
       std::ofstream file("data.txt");
       file << "data";
   }  // 自動關閉

   // 不好：需要手動關閉
   FILE* file = fopen("data.txt", "w");
   fprintf(file, "data");
   fclose(file);  // 容易忘記
   ```

2. **始終檢查檔案是否成功開啟**
   ```cpp
   std::ifstream file("data.txt");
   if (!file.is_open()) {
       std::cerr << "無法開啟檔案\n";
       return;
   }
   ```

3. **使用適當的開啟模式**
   ```cpp
   // 文字檔
   std::ifstream text("data.txt");

   // 二進位檔
   std::ifstream binary("data.bin", std::ios::binary);

   // 附加模式
   std::ofstream append("log.txt", std::ios::app);
   ```

4. **使用 C++17 filesystem 進行檔案操作**
   ```cpp
   namespace fs = std::filesystem;

   // 檢查存在性
   if (fs::exists("file.txt")) {
       // 處理檔案
   }

   // 安全地複製檔案
   fs::copy_file("src.txt", "dst.txt",
                 fs::copy_options::overwrite_existing);
   ```

5. **處理例外**
   ```cpp
   try {
       auto size = fs::file_size("file.txt");
   } catch (const fs::filesystem_error& e) {
       std::cerr << "錯誤: " << e.what() << "\n";
   }
   ```

### DON'T（不應該做的）

1. **不要忘記檢查錯誤**
   ```cpp
   // 不好
   std::ifstream file("data.txt");
   std::string data;
   file >> data;  // 如果開啟失敗會怎樣？

   // 好
   std::ifstream file("data.txt");
   if (file) {
       std::string data;
       file >> data;
   }
   ```

2. **不要在二進位模式和文字模式間混淆**
   ```cpp
   // 不好：用文字模式處理二進位資料
   std::ofstream file("data.bin");  // 預設是文字模式
   file.write(buffer, size);  // 可能會有問題

   // 好：明確指定二進位模式
   std::ofstream file("data.bin", std::ios::binary);
   file.write(buffer, size);
   ```

3. **不要假設檔案一定存在**
   ```cpp
   // 不好
   auto size = fs::file_size("file.txt");  // 如果不存在會拋出例外

   // 好
   if (fs::exists("file.txt")) {
       auto size = fs::file_size("file.txt");
   }
   ```

4. **不要使用過時的 C-style API（如果可以避免）**
   ```cpp
   // 不建議
   FILE* f = fopen("file.txt", "r");
   // ...
   fclose(f);

   // 建議
   std::ifstream file("file.txt");
   ```

5. **不要在跨平台程式中使用硬編碼的路徑分隔符**
   ```cpp
   // 不好
   std::string path = "C:\\Users\\name\\file.txt";  // 只在 Windows 上有效

   // 好：使用 filesystem
   fs::path p = fs::path("C:") / "Users" / "name" / "file.txt";
   ```

## 小結

**C++ 檔案處理的核心概念：**

1. **三種主要方式**：
   - **C-style (FILE*)**：效能好，但需手動管理
   - **C++ streams (fstream)**：RAII、型別安全、推薦使用
   - **C++17 filesystem**：跨平台檔案系統操作

2. **檔案串流類別**：
   - `ifstream`：讀取檔案
   - `ofstream`：寫入檔案
   - `fstream`：讀寫檔案

3. **重要概念**：
   - 始終檢查檔案是否成功開啟
   - 使用 RAII 自動管理資源
   - 區分文字模式和二進位模式
   - 適當的錯誤處理

4. **現代 C++ 建議**：
   - 優先使用 `fstream`（C++98）
   - 使用 `filesystem` 進行檔案操作（C++17）
   - 避免使用 C-style FILE*（除非有特殊需求）

5. **最佳實踐**：
   - 使用適當的開啟模式
   - 處理所有可能的錯誤
   - 使用 filesystem 進行路徑處理
   - 跨平台考量

**記住：現代 C++ 提供了強大且安全的檔案處理工具，善用 RAII 和 filesystem 可以讓檔案處理變得簡單且不易出錯！**