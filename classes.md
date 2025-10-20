### Design a Stack (with getMin)

```c++
#include <algorithm>
#include <iostream>
#include <vector>

class Stack
{
public:
  void push(int val)
  {
    data.push_back(val);

    if (minValues.empty() || minValues.back() >= val)
    {
      minValues.push_back(val);
    }
  }

  void pop()
  {
     if(!data.empty())
     {
      if (data.back() == minValues.back()) minValues.pop_back();
      data.pop_back();
     }
  }

  int top() const
  {
    if (data.empty()) throw std::out_of_range("Stack is empty");
    return data.back();
  }

  int getMin() const
  {
    if (minValues.empty()) throw std::out_of_range("Stack is empty");
    return minValues.back();
  }

private:
  std::vector<int> data {};
  std::vector<int> minValues {};
};

// To execute C++, please define "int main()"
int main() {

  return 0;
}

```

### Design a logger rate limited

```c++
class Logger
{
public:
  bool shouldPrintMessage(int timestamp, const std::string& message)
  {
    if (map.contains(message))
    {
      const auto lastPrintTime { map.at(message) };
      if (timestamp - lastPrintTime < 10)
      {
        return false;
      }
      else
      {
        map[message] = timestamp;
        return true;
      }
    }

    map[message] = timestamp;
    return true;
  }

private:
  std::unordered_map<std::string, int> map {};
};
```

```c++
class Logger
{
public:
  bool shouldPrintMessage(int timestamp, const std::string& message)
  {
    while (!messages.empty() && timestamp - messages.front().first >= 10)
    {
      const auto& [time, mes] = messages.front();
      map.erase(mes);
      messages.pop();
    }

    if (map.contains(message))
    {
        return false;
    }

    map[message] = timestamp;
    messages.emplace(timestamp, message);
    return true;
  }

private:
  std::unordered_map<std::string, int> map {};
  std::queue<std::pair<int, std::string>> messages {};
};
```

```c++
class LRUCache
{
public:
  LRUCache(int capacity) : capactiy_(capacity) {};

  int get(int key)
  {
    if (!map.contains(key)) return -1;
    dll.splice(dll.begin(),dll,map[key]);
    return map.at(key)->second;
  }

  void put(int key, int value)
  {
    if (map.contains(key))
    {
      dll.splice(dll.begin(), dll, map[key]);
      map[key]->second = value;
      return;
    }
    if (static_cast<int>(dll.size()) == capactiy_)
    {
      const auto& keyToDelete { dll.back().first };
      dll.pop_back();
      map.erase(keyToDelete);
    }
    dll.emplace_front(std::pair{key, value});
    map[key] = dll.begin();
  }

private:
int capactiy_ {};
list<std::pair<int, int>> dll {};
std::unordered_map<int, list<std::pair<int, int>>::iterator> map {};
};

```

# PlayTracker

```c++
class PlayTracker
{
public:
  void recordPlay(const std::string& songId)
  {
    if (songId.empty()) return;

    map[songId]++;

    if (mostPlayedId.empty() || map[mostPlayedId] < map[songId])
    {
      mostPlayedId = songId;
    }
  }

  std::string getMostPlayed() const
  {
    return mostPlayedId;
  }

  int getPlayCount(const std::string& songId) const
  {
    if (map.contains(songId)) return map.at(songId);

    return -1;
  }

private:
  std::string mostPlayedId { "" };
  std::unordered_map<std::string, int> map {};
};
```

```c++
#include <string>
#include <unordered_map>
#include <queue>
#include <utility>

class PlayTracker
{
public:
    explicit PlayTracker(int maxHistory) : maxHistory_(maxHistory) {}

    // Record a song play.
    void recordPlay(const std::string& songId)
    {
        if (songId.empty()) return;

        // Add new play.
        playQueue_.push(songId);
        playCount_[songId]++;

        // Remove oldest if over capacity.
        if (static_cast<int>(playQueue_.size()) > maxHistory_)
        {
            const std::string& oldest = playQueue_.front();
            playQueue_.pop();
            if (--playCount_[oldest] == 0)
                playCount_.erase(oldest);
        }

        // Optionally update mostPlayedId_ lazily for O(1) retrieval.
        if (mostPlayedId_.empty() || playCount_[songId] >= playCount_[mostPlayedId_])
            mostPlayedId_ = songId;
    }

    // Get the most-played song in the current window.
    std::string getMostPlayed() const
    {
        return mostPlayedId_;
    }

    // Get how many times a song appears in the current window.
    int getPlayCount(const std::string& songId) const
    {
        if (auto it = playCount_.find(songId); it != playCount_.end())
            return it->second;
        return 0;
    }

private:
    int maxHistory_;                                       // max number of plays to retain
    std::queue<std::string> playQueue_;                    // recent plays
    std::unordered_map<std::string, int> playCount_;       // songId → count
    std::string mostPlayedId_;                             // cached most played song
};

```
