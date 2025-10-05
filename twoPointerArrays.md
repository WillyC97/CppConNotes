```c++
std::vector<int> twoSum(const std::vector<int>& numbers, int target)
{
  int leftPtr {};
  int rightPtr { static_cast<int>(numbers.size()) - 1};

  while (leftPtr < rightPtr)
  {
    // std::cout << "Left :" << numbers[leftPtr] << std::endl;
    // std::cout << "Right :" << numbers[rightPtr] << std::endl;

    const auto currentSum { numbers[leftPtr] + numbers[rightPtr] };
    if (currentSum == target) return { leftPtr + 1, rightPtr + 1};
    if (currentSum > target) rightPtr--;
    if (currentSum < target) leftPtr++;
  }

  return {};
}
```

```c++
int removeDuplicates(std::vector<int>& nums)
{
  // nums.erase(std::unique(nums.begin(), nums.end()), nums.end());

  int nextUniqueValuePos {};

  if (nums.empty()) return {};

  for (int i {}; i < static_cast<int>(nums.size()); i++)
  {
    if (nums[nextUniqueValuePos] != nums[i])
    {
      nextUniqueValuePos++;
      nums[nextUniqueValuePos] = nums[i];
    }
  }

  return nextUniqueValuePos + 1;
}
```

```c++
void moveZeroes(std::vector<int>& nums)
{
// int before = nums.size();
// std::erase(nums, 0);                       // erase all zeros
// int removed = before - nums.size();
// nums.insert(nums.end(), removed, 0);

  int lastNonZeroFoundAt {};

  for (int i {}; i < static_cast<int>(nums.size()); i++)
  {
      if (nums[i] != 0)
      {
          nums[lastNonZeroFoundAt] = nums[i];
          lastNonZeroFoundAt = i;
      }
  }

  for (int i = lastNonZeroFoundAt; i < static_cast<int>(nums.size()); i++)
      nums[i] = 0;
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
