## Laboratory work V

Данная лабораторная работа посвещена изучению фреймворков для тестирования на примере **GTest**

[![Coverage Status](https://coveralls.io/repos/github/Lerra227/lab05-fix/badge.svg?branch=refs/heads/master)](https://coveralls.io/github/Lerra227/lab05-fix?branch=refs/heads/master)

```sh
lera@Lerra:~$ mkdir lab05-fix
lera@Lerra:~$ cd lab05-fix
lera@Lerra:~/lab05-fix$ git clone https://github.com/tp-labs/lab05
Cloning into 'lab05'...
remote: Enumerating objects: 137, done.
remote: Counting objects: 100% (25/25), done.
remote: Compressing objects: 100% (9/9), done.
remote: Total 137 (delta 18), reused 16 (delta 16), pack-reused 112
Receiving objects: 100% (137/137), 918.92 KiB | 3.62 MiB/s, done.
Resolving deltas: 100% (60/60), done.
lera@Lerra:~/lab05-fix$ git remote remove origin
lera@Lerra:~/lab05-fix$ git remote add origin https://github.com/Lerra227/lab05-fix
```
```sh
lera@Lerra:~/lab05-fix/lab05$ git submodule add  https://github.com/google/googletest.git
Cloning into '/home/lera/lab05-fix/lab05/googletest'...
remote: Enumerating objects: 26376, done.
remote: Counting objects: 100% (274/274), done.
remote: Compressing objects: 100% (146/146), done.
remote: Total 26376 (delta 145), reused 199 (delta 116), pack-reused 26102
Receiving objects: 100% (26376/26376), 12.51 MiB | 8.09 MiB/s, done.
Resolving deltas: 100% (19456/19456), done.
```
```sh
lera@Lerra:~/lab05-fix/lab05$ cd banking
lera@Lerra:~/lab05-fix/lab05/banking$ rm CMakeList.txt
lera@Lerra:~/lab05-fix/lab05/banking$ nano CMakeLists.txt
```
```sh
cmake_minimum_required(VERSION 3.4)
project(bank_lib)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
add_library(banking STATIC Account.cpp Account.h Transaction.cpp Transaction.h)
```
```sh
lera@Lerra:~/lab05-fix/lab05$ mkdir .github
lera@Lerra:~/lab05-fix/lab05$ cd .github
lera@Lerra:~/lab05-fix/lab05/.github$ mkdir workflows
lera@Lerra:~/lab05-fix/lab05/.github$  cd workflows
lera@Lerra:~/lab05-fix/lab05/.github/workflows$ nano cmake.yml
```
```sh
name: CMake
on:
 push:
  branches: [master]
 pull_request:
  branches: [master]
jobs:
 build_Linux:
  runs-on: ubuntu-latest
  steps:
  - uses: actions/checkout@v3
  - name: Adding gtest
    run: git clone https://github.com/google/googletest.git third-party/gtest -b release-1.11.0
  - name: Install lcov
    run: sudo apt-get install -y lcov
  - name: Config banking with tests
    run: cmake -H. -B ${{github.workspace}}/build -DBUILD_TESTS=ON
  - name: Build banking
    run: cmake --build ${{github.workspace}}/build
  - name: Run tests
    run: build/check
  - name: Do lcov stuff
    run: lcov -c -d build/CMakeFiles/banking.dir/banking/ --include *.cpp --output-file ./coverage/lcov.info
  - name: Publish to coveralls.io
    uses: coverallsapp/github-action@v1.1.2
    with:
      github-token: ${{ secrets.GITHUB_TOKEN }}
```
```sh
lera@Lerra:~/lab05-fix/lab05/.github/workflows$ cd ..
lera@Lerra:~/lab05-fix/lab05/.github$ cd ..
lera@Lerra:~/lab05-fix/lab05$ nano CMakeLists.txt
```
```sh
cmake_minimum_required(VERSION 3.4)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(BUILD_TESTS "Build tests" OFF)

if(BUILD_TESTS)
  add_compile_options(--coverage)
endif()

project (banking)

add_library(banking STATIC ${CMAKE_CURRENT_SOURCE_DIR}/banking/Transaction.cpp ${CMAKE_CURRENT_SOURCE_DIR}/banking/Account.cpp)
target_include_directories(banking PUBLIC
${CMAKE_CURRENT_SOURCE_DIR}/banking )

target_link_libraries(banking gcov)

if(BUILD_TESTS)
  enable_testing()
  add_subdirectory(third-party/gtest)
  file(GLOB BANKING_TEST_SOURCES tests/*.cpp)
  add_executable(check ${BANKING_TEST_SOURCES})
  target_link_libraries(check banking gtest_main)
  add_test(NAME check COMMAND check)
endif()
```
```sh
lera@Lerra:~/lab05-fix/lab05$ mkdir tests
lera@Lerra:~/lab05-fix/lab05$ cd tests
lera@Lerra:~/lab05-fix/lab05/tests$ nano test_Account.cpp
```
```sh
include <Account.h>
#include <gtest/gtest.h>

TEST(Account, Banking){
	Account test(0,0);
	ASSERT_EQ(test.GetBalance(), 0);
	ASSERT_THROW(test.ChangeBalance(100), std::runtime_error);
	test.Lock();
	ASSERT_NO_THROW(test.ChangeBalance(100));
	ASSERT_EQ(test.GetBalance(), 100);
	ASSERT_THROW(test.Lock(), std::runtime_error);
	test.Unlock();
	ASSERT_THROW(test.ChangeBalance(100), std::runtime_error);
}
```
```sh
lera@Lerra:~/lab05-fix/lab05/tests$ nano test_Transaction.cpp
```
```sh
#include <Account.h>
#include <Transaction.h>
#include <gtest/gtest.h>

TEST(Transaction, Banking){
	const int base_A = 5000, base_B = 5000, base_fee = 100;
	Account Alice(0,base_A), Bob(1,base_B);
	Transaction test_tran;
	ASSERT_EQ(test_tran.fee(), 1);
	test_tran.set_fee(base_fee);
	ASSERT_EQ(test_tran.fee(), base_fee);
	ASSERT_THROW(test_tran.Make(Alice, Alice, 1000), std::logic_error);
	ASSERT_THROW(test_tran.Make(Alice, Bob, -50), std::invalid_argument);
	ASSERT_THROW(test_tran.Make(Alice, Bob, 50), std::logic_error);
	if (test_tran.fee()*2-1 >= 100)
		ASSERT_EQ(test_tran.Make(Alice, Bob, test_tran.fee()*2-1), false);
	Alice.Lock();
	ASSERT_THROW(test_tran.Make(Alice, Bob, 1000), std::runtime_error);
	Alice.Unlock();
	ASSERT_EQ(test_tran.Make(Alice, Bob, 1000), true);
	ASSERT_EQ(Bob.GetBalance(), base_B+1000);
	ASSERT_EQ(Alice.GetBalance(), base_A-1000-base_fee);
	ASSERT_EQ(test_tran.Make(Alice, Bob, 3900), false);
	ASSERT_EQ(Bob.GetBalance(), base_B+1000);
	ASSERT_EQ(Alice.GetBalance(), base_A-1000-base_fee);
}
```
```sh
lera@Lerra:~/lab05-fix/lab05/tests$ cd ..
lera@Lerra:~/lab05-fix/lab05$ mkdir coverage
lera@Lerra:~/lab05-fix/lab05$ cd coverage
lera@Lerra:~/lab05-fix/lab05/coverage$ touch temp.txt
```
```sh
lera@Lerra:~/lab05-fix/lab05/coverage$ cd ..
lera@Lerra:~/lab05-fix/lab05$ cd ..
lera@Lerra:~/lab05-fix$ cd lab05
lera@Lerra:~/lab05-fix/lab05$ git add -A
lera@Lerra:~/lab05-fix/lab05$ git commit -m "second"
[master 9fe6970] second
 5 files changed, 106 insertions(+)
 create mode 100644 .github/workflows/cmake.yml
 create mode 100644 CMakeLists.txt
 create mode 100644 coverage/temp.txt
 create mode 100644 tests/test_Account.cpp
 create mode 100644 tests/test_Transaction.cpp
lera@Lerra:~/lab05-fix/lab05$ git push origin master
Username for 'https://github.com': Lerra227
Password for 'https://Lerra227@github.com':
Enumerating objects: 12, done.
Counting objects: 100% (12/12), done.
Delta compression using up to 20 threads
Compressing objects: 100% (7/7), done.
Writing objects: 100% (11/11), 1.99 KiB | 1.99 MiB/s, done.
Total 11 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To https://github.com/Lerra227/lab05-fix
   f213d9f..9fe6970  master -> master
```
```
Copyright (c) 2015-2021 The ISC Authors
```
