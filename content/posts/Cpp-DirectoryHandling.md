+++
date = '2025-12-17T00:01:29+08:00'
draft = false
title = '[C++] 目錄處理：C++17 filesystem 完整指南'
categories = ['C++ Notes']
tags = ['C++', 'C++17', 'filesystem']
+++

## 前言

在 C++17 之前，目錄操作需要依賴平台特定的 API（如 Windows 的 `_mkdir`、POSIX 的 `opendir`）或第三方函式庫（如 Boost.Filesystem）。C++17 引入的 `<filesystem>` 標準函式庫提供了跨平台、易用的目錄處理功能。

本文將介紹如何使用 C++17 filesystem 進行目錄操作，包括建立、刪除、遍歷目錄等常見任務。

## C++17 filesystem 簡介

### 基本設定

```cpp
#include <filesystem>
#include <iostream>

// 為了簡潔，使用命名空間別名
namespace fs = std::filesystem;

int main() {
    // 現在可以使用 fs:: 代替 std::filesystem::
    fs::path p = fs::current_path();
    std::cout << "當前目錄: " << p << "\n";

    return 0;
}
```

### 編譯設定

```bash
# GCC/Clang (C++17)
g++ -std=c++17 program.cpp -o program

# 部分舊版 GCC 可能需要連結 filesystem 函式庫
g++ -std=c++17 program.cpp -o program -lstdc++fs

# MSVC
cl /EHsc /std:c++17 program.cpp
```

## 目錄的基本操作

### 取得當前目錄

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

void getCurrentDirectory() {
    // 取得當前工作目錄
    fs::path cwd = fs::current_path();
    std::cout << "當前目錄: " << cwd << "\n";

    // 以字串形式取得
    std::string cwdStr = cwd.string();
    std::cout << "字串形式: " << cwdStr << "\n";
}
```

### 變更當前目錄

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

void changeDirectory() {
    std::cout << "原始目錄: " << fs::current_path() << "\n";

    try {
        // 變更到指定目錄
        fs::current_path("/tmp");
        std::cout << "新目錄: " << fs::current_path() << "\n";

        // 回到上一層目錄
        fs::current_path("..");
        std::cout << "上一層目錄: " << fs::current_path() << "\n";

    } catch (const fs::filesystem_error& e) {
        std::cerr << "錯誤: " << e.what() << "\n";
    }
}
```

### 檢查目錄是否存在

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

void checkDirectoryExists() {
    std::string dirName = "my_directory";

    // 方法 1：檢查是否存在
    if (fs::exists(dirName)) {
        std::cout << "目錄存在\n";
    } else {
        std::cout << "目錄不存在\n";
    }

    // 方法 2：檢查是否存在且為目錄
    if (fs::exists(dirName) && fs::is_directory(dirName)) {
        std::cout << "這是一個目錄\n";
    }

    // 方法 3：使用 status
    auto status = fs::status(dirName);
    if (fs::is_directory(status)) {
        std::cout << "確認是目錄\n";
    }
}
```

### 建立目錄

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

void createDirectories() {
    // 1. 建立單一目錄
    if (fs::create_directory("new_folder")) {
        std::cout << "目錄建立成功\n";
    } else {
        std::cout << "目錄已存在或建立失敗\n";
    }

    // 2. 建立巢狀目錄（遞迴建立）
    if (fs::create_directories("parent/child/grandchild")) {
        std::cout << "巢狀目錄建立成功\n";
    } else {
        std::cout << "目錄已存在或建立失敗\n";
    }

    // 3. 錯誤處理
    try {
        fs::create_directories("path/to/nested/folder");
        std::cout << "目錄建立成功\n";
    } catch (const fs::filesystem_error& e) {
        std::cerr << "建立失敗: " << e.what() << "\n";
    }
}
```

### 刪除目錄

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

void removeDirectories() {
    // 1. 刪除空目錄
    if (fs::remove("empty_folder")) {
        std::cout << "目錄已刪除\n";
    } else {
        std::cout << "目錄不存在或刪除失敗\n";
    }

    // 2. 遞迴刪除目錄（包含所有內容）
    try {
        auto count = fs::remove_all("folder_with_contents");
        std::cout << "刪除了 " << count << " 個項目\n";
    } catch (const fs::filesystem_error& e) {
        std::cerr << "刪除失敗: " << e.what() << "\n";
    }

    // 警告：remove_all 非常危險，務必小心使用
}
```

### 重新命名/移動目錄

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

void renameDirectory() {
    try {
        // 重新命名目錄
        fs::rename("old_name", "new_name");
        std::cout << "目錄重新命名成功\n";

        // 移動目錄到其他位置
        fs::rename("new_name", "another_folder/new_name");
        std::cout << "目錄移動成功\n";

    } catch (const fs::filesystem_error& e) {
        std::cerr << "操作失敗: " << e.what() << "\n";
    }
}
```

## 列舉目錄內容

### 列舉目錄（非遞迴）

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

void listDirectory(const fs::path& directory) {
    if (!fs::exists(directory) || !fs::is_directory(directory)) {
        std::cerr << "目錄不存在: " << directory << "\n";
        return;
    }

    std::cout << "目錄內容: " << directory << "\n";
    std::cout << std::string(50, '-') << "\n";

    try {
        // 使用 directory_iterator 遍歷目錄
        for (const auto& entry : fs::directory_iterator(directory)) {
            std::cout << entry.path().filename() << "\t";

            if (fs::is_directory(entry)) {
                std::cout << "<DIR>";
            } else if (fs::is_regular_file(entry)) {
                std::cout << fs::file_size(entry) << " bytes";
            }

            std::cout << "\n";
        }
    } catch (const fs::filesystem_error& e) {
        std::cerr << "錯誤: " << e.what() << "\n";
    }
}

int main() {
    listDirectory(".");  // 列舉當前目錄
    return 0;
}
```

### 列舉目錄（遞迴）

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

void listDirectoryRecursive(const fs::path& directory) {
    if (!fs::exists(directory) || !fs::is_directory(directory)) {
        std::cerr << "目錄不存在: " << directory << "\n";
        return;
    }

    std::cout << "遞迴列舉目錄: " << directory << "\n";
    std::cout << std::string(50, '-') << "\n";

    try {
        // 使用 recursive_directory_iterator 遞迴遍歷
        for (const auto& entry : fs::recursive_directory_iterator(directory)) {
            // 計算縮排（根據深度）
            auto depth = std::distance(directory.begin(), entry.path().begin()) - 1;
            std::string indent(depth * 2, ' ');

            std::cout << indent << entry.path().filename();

            if (fs::is_directory(entry)) {
                std::cout << "/";
            } else if (fs::is_regular_file(entry)) {
                std::cout << " (" << fs::file_size(entry) << " bytes)";
            }

            std::cout << "\n";
        }
    } catch (const fs::filesystem_error& e) {
        std::cerr << "錯誤: " << e.what() << "\n";
    }
}
```

### 過濾特定檔案類型

```cpp
#include <filesystem>
#include <iostream>
#include <vector>
#include <algorithm>

namespace fs = std::filesystem;

// 尋找特定副檔名的檔案
std::vector<fs::path> findFilesByExtension(
    const fs::path& directory,
    const std::string& extension)
{
    std::vector<fs::path> result;

    if (!fs::exists(directory) || !fs::is_directory(directory)) {
        return result;
    }

    try {
        for (const auto& entry : fs::recursive_directory_iterator(directory)) {
            if (fs::is_regular_file(entry) &&
                entry.path().extension() == extension)
            {
                result.push_back(entry.path());
            }
        }
    } catch (const fs::filesystem_error& e) {
        std::cerr << "錯誤: " << e.what() << "\n";
    }

    return result;
}

int main() {
    // 尋找所有 .cpp 檔案
    auto cppFiles = findFilesByExtension(".", ".cpp");

    std::cout << "找到 " << cppFiles.size() << " 個 .cpp 檔案:\n";
    for (const auto& file : cppFiles) {
        std::cout << "  " << file << "\n";
    }

    // 尋找所有 .txt 檔案
    auto txtFiles = findFilesByExtension(".", ".txt");

    std::cout << "\n找到 " << txtFiles.size() << " 個 .txt 檔案:\n";
    for (const auto& file : txtFiles) {
        std::cout << "  " << file << "\n";
    }

    return 0;
}
```

## 目錄資訊查詢

### 取得目錄大小

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

// 計算目錄總大小（包含子目錄）
std::uintmax_t getDirectorySize(const fs::path& directory) {
    std::uintmax_t size = 0;

    if (!fs::exists(directory) || !fs::is_directory(directory)) {
        return 0;
    }

    try {
        for (const auto& entry : fs::recursive_directory_iterator(directory)) {
            if (fs::is_regular_file(entry)) {
                size += fs::file_size(entry);
            }
        }
    } catch (const fs::filesystem_error& e) {
        std::cerr << "錯誤: " << e.what() << "\n";
    }

    return size;
}

// 格式化檔案大小
std::string formatSize(std::uintmax_t size) {
    const char* units[] = {"B", "KB", "MB", "GB", "TB"};
    int unitIndex = 0;
    double displaySize = static_cast<double>(size);

    while (displaySize >= 1024.0 && unitIndex < 4) {
        displaySize /= 1024.0;
        unitIndex++;
    }

    char buffer[50];
    snprintf(buffer, sizeof(buffer), "%.2f %s", displaySize, units[unitIndex]);
    return buffer;
}

int main() {
    auto size = getDirectorySize(".");
    std::cout << "目錄大小: " << size << " bytes\n";
    std::cout << "格式化大小: " << formatSize(size) << "\n";

    return 0;
}
```

### 統計目錄內容

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

struct DirectoryStats {
    size_t fileCount = 0;
    size_t dirCount = 0;
    std::uintmax_t totalSize = 0;
};

DirectoryStats analyzeDirectory(const fs::path& directory) {
    DirectoryStats stats;

    if (!fs::exists(directory) || !fs::is_directory(directory)) {
        return stats;
    }

    try {
        for (const auto& entry : fs::recursive_directory_iterator(directory)) {
            if (fs::is_regular_file(entry)) {
                stats.fileCount++;
                stats.totalSize += fs::file_size(entry);
            } else if (fs::is_directory(entry)) {
                stats.dirCount++;
            }
        }
    } catch (const fs::filesystem_error& e) {
        std::cerr << "錯誤: " << e.what() << "\n";
    }

    return stats;
}

int main() {
    auto stats = analyzeDirectory(".");

    std::cout << "目錄分析結果:\n";
    std::cout << "  檔案數量: " << stats.fileCount << "\n";
    std::cout << "  目錄數量: " << stats.dirCount << "\n";
    std::cout << "  總大小: " << stats.totalSize << " bytes\n";

    return 0;
}
```

## 實用範例

### 範例 1：目錄備份工具

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

class DirectoryBackup {
public:
    // 備份整個目錄
    static bool backup(const fs::path& source, const fs::path& destination) {
        if (!fs::exists(source) || !fs::is_directory(source)) {
            std::cerr << "來源目錄不存在: " << source << "\n";
            return false;
        }

        try {
            // 建立目標目錄
            fs::create_directories(destination);

            // 遞迴複製所有內容
            for (const auto& entry : fs::recursive_directory_iterator(source)) {
                const auto& path = entry.path();
                auto relativePath = fs::relative(path, source);
                auto destinationPath = destination / relativePath;

                if (fs::is_directory(path)) {
                    fs::create_directories(destinationPath);
                    std::cout << "建立目錄: " << destinationPath << "\n";
                } else if (fs::is_regular_file(path)) {
                    fs::copy_file(path, destinationPath,
                                fs::copy_options::overwrite_existing);
                    std::cout << "複製檔案: " << destinationPath << "\n";
                }
            }

            std::cout << "備份完成!\n";
            return true;

        } catch (const fs::filesystem_error& e) {
            std::cerr << "備份失敗: " << e.what() << "\n";
            return false;
        }
    }
};

int main() {
    DirectoryBackup::backup("my_project", "my_project_backup");
    return 0;
}
```

### 範例 2：清理臨時檔案

```cpp
#include <filesystem>
#include <iostream>
#include <vector>
#include <chrono>

namespace fs = std::filesystem;

class TempFileCleaner {
public:
    // 刪除舊的臨時檔案
    static size_t cleanOldFiles(const fs::path& directory, int daysOld) {
        size_t deletedCount = 0;

        if (!fs::exists(directory) || !fs::is_directory(directory)) {
            return 0;
        }

        // 計算截止時間
        auto now = fs::file_time_type::clock::now();
        auto threshold = now - std::chrono::hours(24 * daysOld);

        try {
            for (const auto& entry : fs::directory_iterator(directory)) {
                if (!fs::is_regular_file(entry)) {
                    continue;
                }

                auto lastModified = fs::last_write_time(entry);

                if (lastModified < threshold) {
                    std::cout << "刪除舊檔案: " << entry.path() << "\n";
                    fs::remove(entry);
                    deletedCount++;
                }
            }
        } catch (const fs::filesystem_error& e) {
            std::cerr << "錯誤: " << e.what() << "\n";
        }

        return deletedCount;
    }

    // 刪除大型檔案
    static size_t cleanLargeFiles(const fs::path& directory, std::uintmax_t sizeThreshold) {
        size_t deletedCount = 0;

        if (!fs::exists(directory) || !fs::is_directory(directory)) {
            return 0;
        }

        try {
            for (const auto& entry : fs::recursive_directory_iterator(directory)) {
                if (!fs::is_regular_file(entry)) {
                    continue;
                }

                auto fileSize = fs::file_size(entry);

                if (fileSize > sizeThreshold) {
                    std::cout << "刪除大型檔案: " << entry.path()
                             << " (" << fileSize << " bytes)\n";
                    fs::remove(entry);
                    deletedCount++;
                }
            }
        } catch (const fs::filesystem_error& e) {
            std::cerr << "錯誤: " << e.what() << "\n";
        }

        return deletedCount;
    }
};

int main() {
    // 刪除 7 天以前的檔案
    auto count1 = TempFileCleaner::cleanOldFiles("/tmp", 7);
    std::cout << "刪除了 " << count1 << " 個舊檔案\n";

    // 刪除大於 100MB 的檔案
    auto count2 = TempFileCleaner::cleanLargeFiles("/tmp", 100 * 1024 * 1024);
    std::cout << "刪除了 " << count2 << " 個大型檔案\n";

    return 0;
}
```

### 範例 3：目錄同步工具

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

class DirectorySync {
public:
    // 將來源目錄同步到目標目錄
    static bool sync(const fs::path& source, const fs::path& destination) {
        if (!fs::exists(source) || !fs::is_directory(source)) {
            std::cerr << "來源目錄不存在\n";
            return false;
        }

        try {
            // 建立目標目錄
            fs::create_directories(destination);

            // 複製新檔案和更新的檔案
            syncFiles(source, destination);

            // 刪除目標目錄中多餘的檔案
            removeExtraFiles(source, destination);

            std::cout << "同步完成!\n";
            return true;

        } catch (const fs::filesystem_error& e) {
            std::cerr << "同步失敗: " << e.what() << "\n";
            return false;
        }
    }

private:
    static void syncFiles(const fs::path& source, const fs::path& destination) {
        for (const auto& entry : fs::recursive_directory_iterator(source)) {
            const auto& srcPath = entry.path();
            auto relativePath = fs::relative(srcPath, source);
            auto dstPath = destination / relativePath;

            if (fs::is_directory(srcPath)) {
                fs::create_directories(dstPath);
            } else if (fs::is_regular_file(srcPath)) {
                bool shouldCopy = false;

                if (!fs::exists(dstPath)) {
                    shouldCopy = true;
                    std::cout << "新檔案: " << relativePath << "\n";
                } else {
                    auto srcTime = fs::last_write_time(srcPath);
                    auto dstTime = fs::last_write_time(dstPath);

                    if (srcTime > dstTime) {
                        shouldCopy = true;
                        std::cout << "更新檔案: " << relativePath << "\n";
                    }
                }

                if (shouldCopy) {
                    fs::copy_file(srcPath, dstPath,
                                fs::copy_options::overwrite_existing);
                }
            }
        }
    }

    static void removeExtraFiles(const fs::path& source, const fs::path& destination) {
        for (const auto& entry : fs::recursive_directory_iterator(destination)) {
            const auto& dstPath = entry.path();
            auto relativePath = fs::relative(dstPath, destination);
            auto srcPath = source / relativePath;

            if (!fs::exists(srcPath)) {
                std::cout << "刪除多餘項目: " << relativePath << "\n";
                if (fs::is_directory(dstPath)) {
                    fs::remove_all(dstPath);
                } else {
                    fs::remove(dstPath);
                }
            }
        }
    }
};

int main() {
    DirectorySync::sync("source_folder", "destination_folder");
    return 0;
}
```

### 範例 4：尋找重複檔案

```cpp
#include <filesystem>
#include <iostream>
#include <fstream>
#include <map>
#include <vector>
#include <sstream>
#include <iomanip>

namespace fs = std::filesystem;

class DuplicateFinder {
public:
    // 尋找重複檔案（基於大小和內容）
    static void findDuplicates(const fs::path& directory) {
        std::map<std::uintmax_t, std::vector<fs::path>> sizeGroups;

        // 第一步：按檔案大小分組
        for (const auto& entry : fs::recursive_directory_iterator(directory)) {
            if (fs::is_regular_file(entry)) {
                auto size = fs::file_size(entry);
                sizeGroups[size].push_back(entry.path());
            }
        }

        // 第二步：檢查大小相同的檔案內容是否相同
        for (const auto& [size, files] : sizeGroups) {
            if (files.size() < 2) {
                continue;  // 只有一個檔案，跳過
            }

            std::map<std::string, std::vector<fs::path>> hashGroups;

            for (const auto& file : files) {
                auto hash = computeFileHash(file);
                hashGroups[hash].push_back(file);
            }

            // 輸出重複檔案
            for (const auto& [hash, duplicates] : hashGroups) {
                if (duplicates.size() > 1) {
                    std::cout << "\n找到重複檔案（大小: " << size << " bytes）:\n";
                    for (const auto& dup : duplicates) {
                        std::cout << "  " << dup << "\n";
                    }
                }
            }
        }
    }

private:
    // 簡單的檔案雜湊（實際應使用 MD5 或 SHA256）
    static std::string computeFileHash(const fs::path& file) {
        std::ifstream ifs(file, std::ios::binary);
        if (!ifs) {
            return "";
        }

        std::ostringstream oss;
        oss << ifs.rdbuf();
        std::string content = oss.str();

        // 簡單的雜湊（不安全，僅作示範）
        std::hash<std::string> hasher;
        size_t hash = hasher(content);

        std::ostringstream hashStream;
        hashStream << std::hex << hash;
        return hashStream.str();
    }
};

int main() {
    DuplicateFinder::findDuplicates(".");
    return 0;
}
```

## 跨平台考量

### 路徑分隔符

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

void pathSeparators() {
    // 好：使用 / 運算子（跨平台）
    fs::path p1 = fs::path("folder") / "subfolder" / "file.txt";
    std::cout << "路徑 1: " << p1 << "\n";

    // 不好：硬編碼分隔符（不跨平台）
    std::string p2 = "folder\\subfolder\\file.txt";  // 只在 Windows 上有效

    // filesystem 會自動處理分隔符
    fs::path p3 = "folder/subfolder/file.txt";  // 在所有平台上都有效
    std::cout << "路徑 3: " << p3 << "\n";

    // 取得系統的路徑分隔符
    std::cout << "路徑分隔符: " << fs::path::preferred_separator << "\n";
}
```

### 處理權限問題

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

void handlePermissions() {
    fs::path testDir = "test_permissions";

    try {
        // 建立目錄
        fs::create_directory(testDir);

        // 設定權限（POSIX 系統）
        fs::permissions(testDir,
                       fs::perms::owner_all | fs::perms::group_read,
                       fs::perm_options::replace);

        std::cout << "權限設定成功\n";

    } catch (const fs::filesystem_error& e) {
        // Windows 上可能不支援某些權限操作
        std::cerr << "權限操作失敗: " << e.what() << "\n";
    }
}
```

## 最佳實踐

### DO（應該做的）

1. **始終使用 filesystem 的 path 類別**
   ```cpp
   // 好：使用 fs::path
   fs::path p = fs::current_path() / "folder" / "file.txt";

   // 不好：手動串接字串
   std::string p2 = getCurrentPath() + "/folder/file.txt";
   ```

2. **檢查目錄是否存在**
   ```cpp
   if (fs::exists(directory) && fs::is_directory(directory)) {
       // 處理目錄
   }
   ```

3. **使用 try-catch 處理例外**
   ```cpp
   try {
       fs::remove_all("folder");
   } catch (const fs::filesystem_error& e) {
       std::cerr << "錯誤: " << e.what() << "\n";
   }
   ```

4. **使用範圍 for 迴圈遍歷目錄**
   ```cpp
   for (const auto& entry : fs::directory_iterator(dir)) {
       std::cout << entry.path() << "\n";
   }
   ```

5. **使用 / 運算子組合路徑**
   ```cpp
   fs::path fullPath = baseDir / subDir / filename;
   ```

### DON'T（不應該做的）

1. **不要硬編碼路徑分隔符**
   ```cpp
   // 不好
   std::string path = "C:\\Users\\name\\file.txt";  // 不跨平台

   // 好
   fs::path p = fs::path("C:") / "Users" / "name" / "file.txt";
   ```

2. **不要假設目錄一定存在**
   ```cpp
   // 不好
   for (const auto& entry : fs::directory_iterator(dir)) {
       // 如果 dir 不存在會拋出例外
   }

   // 好
   if (fs::exists(dir) && fs::is_directory(dir)) {
       for (const auto& entry : fs::directory_iterator(dir)) {
           // 安全
       }
   }
   ```

3. **不要忘記處理 remove_all 的危險性**
   ```cpp
   // 危險！務必確認路徑正確
   fs::remove_all(userInputPath);  // 可能刪除重要資料

   // 好：先確認
   if (isSafeToDelete(userInputPath)) {
       fs::remove_all(userInputPath);
   }
   ```

4. **不要在遞迴操作中修改正在遍歷的目錄**
   ```cpp
   // 危險
   for (const auto& entry : fs::recursive_directory_iterator(dir)) {
       fs::remove(entry);  // 可能導致迭代器失效
   }

   // 好：先收集再刪除
   std::vector<fs::path> toDelete;
   for (const auto& entry : fs::recursive_directory_iterator(dir)) {
       toDelete.push_back(entry);
   }
   for (const auto& path : toDelete) {
       fs::remove(path);
   }
   ```

5. **不要忽略符號連結的問題**
   ```cpp
   // 可能造成無限迴圈
   for (const auto& entry : fs::recursive_directory_iterator(dir)) {
       // 如果有符號連結指向父目錄...
   }

   // 好：設定選項
   auto options = fs::directory_options::skip_permission_denied |
                  fs::directory_options::follow_directory_symlink;
   for (const auto& entry : fs::recursive_directory_iterator(dir, options)) {
       // 較安全
   }
   ```

## 小結

**C++17 filesystem 目錄處理的核心概念：**

1. **主要功能**：
   - 建立、刪除、重新命名目錄
   - 遍歷目錄內容（遞迴和非遞迴）
   - 查詢目錄資訊（大小、檔案數量等）
   - 路徑操作和組合

2. **重要類別**：
   - `fs::path`：路徑表示和操作
   - `fs::directory_iterator`：非遞迴目錄遍歷
   - `fs::recursive_directory_iterator`：遞迴目錄遍歷
   - `fs::directory_entry`：目錄項目資訊

3. **常用函式**：
   - `fs::exists()`：檢查存在性
   - `fs::is_directory()`：檢查是否為目錄
   - `fs::create_directory()`：建立單一目錄
   - `fs::create_directories()`：遞迴建立目錄
   - `fs::remove()`：刪除單一項目
   - `fs::remove_all()`：遞迴刪除目錄

4. **最佳實踐**：
   - 始終檢查目錄是否存在
   - 使用 try-catch 處理例外
   - 使用 `fs::path` 進行路徑操作
   - 注意跨平台相容性
   - 小心使用 `remove_all()`

5. **安全考量**：
   - 驗證使用者輸入的路徑
   - 處理權限錯誤
   - 注意符號連結可能造成的問題
   - 避免在遍歷時修改目錄結構

**記住：C++17 filesystem 提供了強大且跨平台的目錄操作功能，善用這些工具可以讓目錄處理變得簡單、安全且可攜！**