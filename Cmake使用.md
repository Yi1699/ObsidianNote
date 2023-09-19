# Windows

## cmake 编译
1. 项目根目录创建build
2. 进入build，执行
```
cmake -G"MinGW Makefiles" ..
```
**运行结果**
```
-- The C compiler identification is GNU 8.1.0
-- The CXX compiler identification is GNU 8.1.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: D:/mingw64/bin/gcc.exe - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: D:/mingw64/bin/c++.exe - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done (2.1s)
-- Generating done (0.0s)
-- Build files have been written to: D:/Graphics/compile_flags/build
```

3. 执行
```
cmake --build .
```
**运行结果**
```
[ 50%] Building CXX object CMakeFiles/cmake_examples_compile_flags.dir/main.cpp.obj
[100%] Linking CXX executable cmake_examples_compile_flags.exe
[100%] Built target cmake_examples_compile_flags
```

4. 执行build内生成的可执行exe文件
例如：
```
.\hellocmake.exe
```

# Linux(Ubuntu)
## Cmake 编译
1. 项目根目录创建build
2. 进入build，执行
```
cmake ..
```
**运行结果**
```
-- The C compiler identification is GNU 8.1.0
-- The CXX compiler identification is GNU 8.1.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: D:/mingw64/bin/gcc.exe - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: D:/mingw64/bin/c++.exe - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done (2.1s)
-- Generating done (0.0s)
-- Build files have been written to: D:/Graphics/compile_flags/build
```

3. 执行
```
make
```
**运行结果**
```
[ 50%] Building CXX object CMakeFiles/optimizer.dir/main.cpp.o
[100%] Linking CXX executable optimizer
[100%] Built target optimizer
```

4. 执行build内生成的可执行文件
例如：
```
./hellocmake
```
