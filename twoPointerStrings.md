```c++
bool validPalindrome(const std::string& str)
{
  // auto revString { str };
  // std::reverse(revString.begin(), revString.end());

  // return str == revString;

  if(str.length() < 2) return true;

  int leftPtr {};
  int rightPtr { static_cast<int>(str.length()) - 1 };

  while (leftPtr < rightPtr)
  {
    while (!isalnum(str[leftPtr])) leftPtr++;
    while(!isalnum(str[rightPtr])) rightPtr--;
    if (str[leftPtr] != str[rightPtr]) return false;

    leftPtr++;
    rightPtr--;
  }

  return true;
}
```

```c++
void reverseString(std::string& str)
{
  if(str.length() < 2) return;

  int leftPtr {};
  int rightPtr { static_cast<int>(str.length()) - 1 };

  while (leftPtr < rightPtr)
  {
    std::swap(str[leftPtr], str[rightPtr]);

    leftPtr++;
    rightPtr--;
  }
}
```

```c++
int palindromeIndex(const std::string& str)
{
  int leftPtr {};
  int rightPtr { static_cast<int>(str.length()) -1 };

  const auto isPalindrome = [](const auto& str, int leftPtr, int rightPtr) -> bool
  {
    while (leftPtr < rightPtr)
    {
      if (str[leftPtr] != str[rightPtr]) return false;
      leftPtr++;
      rightPtr--;
    }

    return true;
  };

  while (leftPtr < rightPtr)
  {
    if (str[leftPtr] != str[rightPtr])
    {
      const auto leftPlusOne { leftPtr + 1};
      const auto rightMinusOne { rightPtr - 1 };
      if (isPalindrome(str, leftPlusOne, rightPtr)) return leftPtr;
      if (isPalindrome(str, leftPtr, rightMinusOne)) return rightPtr;
    }

    leftPtr++;
    rightPtr--;
  }

  return -1;
}
```

```c++
bool isFunny(const std::string& str)
{
  int leftPtr { 1 };
  int rightPtr { static_cast<int>(str.length()) - 1 };

  while (leftPtr < rightPtr)
  {
    if (std::abs(str[leftPtr] - str[leftPtr-1]) != std::abs(str[rightPtr] - str[rightPtr-1])) return false;

    leftPtr++;
    rightPtr --;
  }

  return true;
}
```

```c++
bool isValidSubsequence(const std::vector<int>& array, const std::vector<int>& sequence)
{
  if (sequence.size() > array.size() || sequence.empty()) return false;

  size_t subSequencePtr {};
  for (size_t i {}; i < array.size(); i++)
  {
    if (array[i] == sequence[subSequencePtr])
    {
      subSequencePtr++;
      if(subSequencePtr == sequence.size()) return true;
    }
  }

  return false;
}
```

```c++
std::string caesarCipher(const std::string& str, int shift)
{
    std::string output;

    for (char c : str) {
        if (!isalpha(c))
        {
            output += c;  
            continue;
        }

        char base = isupper(c) ? 'A' : 'a';
        char shifted = c + shift;

        // Wrap forward
        while (shifted > base + 25) shifted -= 26;
        // Wrap backward
        while (shifted < base) shifted += 26;

        output += shifted;
    }

    return output;
}
```

```c++
int strStr(const std::string& haystack, const std::string& needle)
{
  if (needle.empty()) return 0;
  if (needle.size() > haystack.size()) return -1;

  for (size_t hs {}; hs < haystack.length() - needle.length(); hs++)
  {
    size_t n {};

    while (n < needle.size() && haystack[hs + n] == needle[n]) n++;

    if (n == needle.size()) return n;
  }

  return -1;
}
```
