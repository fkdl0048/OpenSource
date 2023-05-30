# 오픈소스SW개론 과제 4 (라이브러리 실습)

18011690 이정안

리눅스 (우분투)에서 C++ 라이브러리를 하나 설치하여 그 라이브러리를 사용하는 예제 코드(C++)를 실행해볼 것

## 과제 설명

- 인터넷에 공개되어있는 C++ 라이브러리 중 원하는 것 하나 선택. 단, 수업자료에 설명되어있는 Eigen, Boost는 제외  

- 리눅스 CLI 명령어로 C++ 예제 소스코드를 컴파일한 후 실행시킬 것. 이 때, 컴파일 시 예제 소스코드가 라이브러리의 헤더파일(X.h)을 참조하거나 라이브러리 파일(XXX.a)을 사용하는 경우여야함. 헤더파일을 참조하거나 라이브러리

- 파일을 링크하는 것을 실습하기 위함임. 해당 라이브러리의 설치 및 예제 소스코드 컴파일/실행 상세 과정을 스크린샷

- 등을 활용하여 작성할 것. 즉, 제출한 문서를 보는 사람이 직접 재현 가능할
정도로 상세히 설명. 다음과 같은 항목이 포함되어야함.

(1) 예제 소스코드(.cpp)

(2) 설치과정. apt-get install 명령어 등을 통해 수행한 경우 사용한 모든 명령어들. 설치를 위해 configure, make, make install 등의 명령어를 사용한 경우 그 명령어도 포함.

(3) 소스코드 컴파일 과정. 컴파일 할 때 사용한 "g++ ..."와 같은 명령어를 작성해야함

(4) 실행 과정 및 실행 결과. “./test" 이런 형태로 생성된 실행파일을 실행할 때
그 명령어를 작성해야함. 또한, 예제 소스코드 실행 결과가 터미널에 출력되었을
때 그 화면을 스크린샷하여 작성

## 선정 라이브러리

- [nana](https://github.com/cnjinhao/nana)

Nana는 개발자가 최신 C++ 스타일로 교차 플랫폼 GUI 응용 프로그램을 쉽게 만들 수 있도록 설계된 C++ 표준과 유사한 GUI 라이브러리입니다.

```cpp
#include <nana/gui.hpp>
#include <nana/gui/widgets/label.hpp>
#include <nana/gui/widgets/button.hpp>

int main()
{
    using namespace nana;

    //Define a form.
    form fm;

    //Define a label and display a text.
    label lab{fm, "Hello, <bold blue size=16>Nana C++ Library</>"};
    lab.format(true);

    //Define a button and answer the click event.
    button btn{fm, "Quit"};
    btn.events().click([&fm]{
        fm.close();
    });

    //Layout management
    fm.div("vert <><<><weight=80% text><>><><weight=24<><button><>><>");
    fm["text"]<<lab;
    fm["button"] << btn;
    fm.collocate();
 
    //Show the form
    fm.show();

    //Start to event loop process, it blocks until the form is closed.
    exec();
}
```

- 가장 표준적으로 실제 nana메인페이지에서 제공하는 예제이다.

더 직관적인 예제

```cpp
//Include nana/gui.hpp header file to enable Nana C++ Library
//for the program.
#include <nana/gui.hpp>
//Include a label widget, we will use it in this example.
#include <nana/gui/widgets/label.hpp>
int main()
{
    //All names of Nana is in the namespace nana;
    using namespace nana;
    //Define a form object, class form will create a window
    //when a form instance is created.
    //The new window default visibility is false.
    form fm;
    //Define a label on the fm(form) with a specified area,
    //and set the caption.
    label lb{ fm, rectangle{ 10, 10, 100, 100 } };
    lb.caption("Hello, world!");
    //Expose the form.
    fm.show();
    //Pass the control of the application to Nana's event
    //service. It blocks the execution for dispatching user
    //input until the form is closed.
    exec();
}
```

- 실제 사용방법은 자바 스윙과 비슷한 방법.

## 설치과정

현재 설치 과정에 대한 실행 환경은 우분투를 WSL2로 설치한 상태이다. (vscode구동)

![image](https://github.com/fkdl0048/OpenSource/assets/84510455/0821db2a-8a61-43c0-94df-e3a72aa4e507)

가장 먼저 `apt`를 사용하여 설치 및 구동환경을 만든다.

```bash
sudo apt update
sudo apt install build-essential
sudo apt install cmake
sudo apt install git
sudo apt install libxft-dev
sudo apt install libxcursor-dev
```

여기서 `libxft-dev`, `libxcursor-dev`는 실행에 필요한 중요한 라이브러리이다.

*없으면 make중에 에러*

그 후, `nana`를 설치한다.

```bash
sudo apt-get update
sudo apt-get install nana
```

*apt와 apt-get은 크게 차이 없다.*

*sudo또한 사용자 권한획득을 위한 인자*

## 소스코드 컴파일 과정

GUI에 관한 오픈소스 라이브러리이다 보니 간단하게 헤더파일 몇가지를 추가한다고 쉽게 컴파일이 되지 않는다.

매우 많은 목적파일과 DLL파일이 필요하다. 이런 내용을 `-I, -L`로 연결할 수 없으니 `CMake`를 사용한다.

사용할 `cpp`파일도 같은 디렉토리에 위치해야 한다.

![Image](https://github.com/fkdl0048/ToDo/assets/84510455/48b6ec77-7ddf-439c-a4a7-74f4f92eb70f)

```cmake
# version 3.12 or later of CMake or needed later for installing nana
cmake_minimum_required(VERSION 3.12-3.18 FATAL_ERROR)

project(nana-cmake
 VERSION 0.1
 LANGUAGES CXX
)

include(FetchContent)

if(MSVC)
  FetchContent_Declare(
    nana
    GIT_REPOSITORY https://github.com/cnjinhao/nana.git
    GIT_TAG        develop-1.8
  )
else()
  FetchContent_Declare(
    nana
    GIT_REPOSITORY https://github.com/cnjinhao/nana.git
    GIT_TAG        master
  )
endif()

FetchContent_GetProperties(nana)
if(NOT nana_POPULATED)
  FetchContent_Populate(nana)
  add_subdirectory(${nana_SOURCE_DIR} ${nana_BINARY_DIR})
endif()

add_executable(hello
 hello.cpp
)

target_link_libraries(hello
 nana
)

target_compile_features(hello 
 PUBLIC 
 cxx_std_17
)
```

- `CMakeLists.txt`파일을 만들고 해당 내용을 넣는다.
- `FetchContent`를 사용하여 `nana`를 다운받는다.
- `add_executable`을 사용하여 실행파일을 만든다.
- `target_link_libraries`를 사용하여 `nana`를 연결한다.
- `target_compile_features`를 사용하여 `c++17`을 사용한다.
- `cmake`를 사용하여 컴파일한다.
- `make`를 사용하여 실행파일을 만든다.
- `./hello`를 사용하여 실행한다.

**cmake 과정**
![Image](https://github.com/fkdl0048/ToDo/assets/84510455/8cf5213b-1561-4c90-b87a-5188c1f9696e)

**make 과정**
![Image](https://github.com/fkdl0048/ToDo/assets/84510455/b9fba2d5-9999-44b2-8c6b-8ab9646fbad5)

![Image](https://github.com/fkdl0048/ToDo/assets/84510455/fa5a3e1c-8c50-4238-ad6c-3e3242f7bd2d)

## 실행결과

```bash
./hello 
```

![Image](https://github.com/fkdl0048/ToDo/assets/84510455/66714a21-e1a1-44c9-9bef-74deb970b8fa)

---  

참고자료

- <https://computingonplains.wordpress.com/creating-a-nana-based-project-using-cmake/>
