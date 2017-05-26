---
layout: episode
title: Hands-on example using PyBind11
teaching: 40
exercises: 0
questions:
  - Write me.
objectives:
  - Unit testing and prototyping of C++ code.
keypoints:
  - Write me.
---

## Section

Assume we are starting a ("Research?") project on  Phonetic Algorithms.
We have received source code for the Soundex algorithm (actually from
 [Jeff Langr's](http://langrsoft.com/about/) book [Modern C++ Programming with Test-Driven Development](http://pragprog.com/book/lotdd/modern-c-programming-with-test-driven-development) Let us say our ambition is to implement
a Phonetic Algorithm class where Soundex is one Phonetic encoding. Other Phonetic
Algorithms we want to implement could be [Daitch-Mokotoff Soundex](https://en.wikipedia.org/wiki/Daitch–Mokotoff_Soundex),
[Cologne phonetics](https://en.wikipedia.org/wiki/Cologne_phonetics) or [NYIIS](https://en.wikipedia.org/wiki/New_York_State_Identification_and_Intelligence_System)

## The Soundex Source code

 Here is source code for a Phonetic Algorithm  [Soundex](https://en.wikipedia.org/wiki/Soundex) . We want to make use of Soundex in Python with help of Pybind11


```C++
#ifndef SOUNDEX_SOUNDEX_H
#define SOUNDEX_SOUNDEX_H
#include <string>
#include <unordered_map>
class Soundex {
    static const size_t MaxCodeLength{4};
public:
    std::string encode(const std::string& word) const {
        return zeroPad(upperFront(head(word))+tail(encodedDigits(word)));
    }

    std::string encodedDigit(char letter) const {
        const std::unordered_map<char,std::string> encodings{
                {'b',"1"},{'f',"1"},{'p',"1"},{'v',"1"},
                {'c',"2"},{'g',"2"},{'j',"2"},{'k',"2"},{'q',"2"},{'s',"2"},{'x',"2"},{'z',"2"},
                {'d',"3"},{'t',"3"},
                {'l',"4"},
                {'m',"5"},{'n',"5"},
                {'r',"6"}

        };
        auto it = encodings.find(lower(letter));
        return it == encodings.end() ? NotADigit: it->second;
    }

private:
    const std::string NotADigit{"*"};

    char lower(char c) const {
        return std::tolower(static_cast<unsigned char>(c));
    }

    std::string head(const std::string& word) const {
        return word.substr(0,1);
    }

    std::string tail(const std::string& word) const {
        return word.substr(1);
    }

    std::string encodedDigits(const std::string& word) const{
        std::string encoding;
        encodeHead(encoding, word);
        encodeTail(encoding,word);
        return encoding;
    }

    void encodeHead(std::string& encoding, const std::string& word) const {
        encoding += encodedDigit(word.front());
    }

    void encodeTail(std::string& encoding, const std::string& word) const{
        for (auto i=1u; i < word.length();i++)  {
            if (!isComplete(encoding))
                encodeLetter(encoding,word[i],word[i-1]);
        }
    }

    void encodeLetter(std::string& encoding, char letter, char lastLetter) const {
        auto digit = encodedDigit(letter);
        if ( digit != NotADigit && ( digit != lastDigit(encoding) || isVowel(lastLetter)))
            encoding += digit;
    }

    bool isComplete(const std::string& encoding) const{
        return encoding.length() == MaxCodeLength;
    }

    std::string zeroPad(const std::string& word) const {
        auto zerosNeeded = MaxCodeLength - word.length();
        return word + std::string(zerosNeeded,'0');
    }

    std::string lastDigit(const std::string& encoding) const {
        if (encoding.empty()) return NotADigit;
        return std::string(1,encoding.back());

    }

    std::string upperFront(const std::string& string) const {
        return std::string(1, std::toupper(static_cast<unsigned char>(string.front())));
    }

    bool isVowel(char letter) const {
        return std::string("aeiouy").find(lower(letter)) != std::string::npos;
    }
};

#endif //SOUNDEX_SOUNDEX_H

```
The source code implements the [Soundex alorithm](https://en.wikipedia.org/wiki/Soundex) which according to Wikipedia maps a name or word to its' first letter
followed by three numerical digits. Outlined the algorithm goes like:
 1. Retain the first letter of the name and drop all other occurences of a,e,i,o,u,y,h,w
 2. Replace consonants with digits as follows (after the firste letter)
    * b,f,p,v -> 1
    * c,g,j,k,q,s,x,z, -> 2
    * d,t -> 3
    * m,n -> 5
    * r -> 6

 3. If two or more letters with the same number are adjacent in the original name
 (before step 1), only retain the first letter; also two letters with the same
 number separated by 'h' or 'w' are coded as a single number, whereas such letters
 separated by a vowel are coded twice. This rule also applies to the first letter.
 4. If you have too few letters in your word that you can't assign three numbers,
 append with zeros until there are three numbers. If you have more than 3 letters,
 just retain the first 3 numbers.


### Building the library using CMake
We will make a subdirectory pybind11 with a *src* subdirectory where put the source code
In addition there will be a CMakeLists.txt file under pybind11 and a CMakeLists.txt
under src:

```shell
pybind11 | -  CMakeLists.txt
         |
	 |
	 | -  src | -  CMakeLists.txt 
	          |
	          | -  Soundex.h
	          |
	          | -  Py11Soundex.cpp

```

Let us create the subdirectory setup and the files:

```shell
(pybind11-example) [lynx@lille-login2~]$ mkdir -p pybind11/src
(pybind11-example) [lynx@lille-login2~]$ cd pybind11/src
(pybind11-example) [lynx@lille-login2~]$ cat > Soundex.h
Create the Soundex.h file by pasting in the source code either by using cat
or your favorite editor.
```
The contents of Py11Soundex.cpp:

```C++
// File: Py11Soundex.cpp
#include <pybind11/pybind11.h>
#include "soundex.h"

namespace py = pybind11;

PYBIND11_PLUGIN(soundex) {
  py::module m("soundex", "pybind11 soundex plugin");

  py::class_<Soundex>(m,"soundex")
    .def(py::init<>())
    .def("encode", &Soundex::encode);
}

``` 

```shell
```
```shell
pybind11-example) [bjornlin@lille-login2 build]$ make
Scanning dependencies of target soundex
[100%] Building CXX object src/CMakeFiles/soundex.dir/Py11soundex.cpp.o
Linking CXX shared module ../lib/soundex.so
[100%] Built target soundex
(pybind11-example) [bjornlin@lille-login2 build]$ cd lib
(pybind11-example) [bjornlin@lille-login2 lib]$ python
Python 2.7.13 |Continuum Analytics, Inc.| (default, Dec 20 2016, 23:09:15) 
[GCC 4.4.7 20120313 (Red Hat 4.4.7-1)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
Anaconda is brought to you by Continuum Analytics.
Please check out: http://continuum.io/thanks and https://anaconda.org
>>> import soundex
>>> p = soundex.soundex()
>>> p.encode('Allison')
u'A425'
>>> 

```