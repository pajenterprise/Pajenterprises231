```cpp
std::vector<XblTitleManagedStatistic> statList;
statList.push_back(XblTitleManagedStatistic{ "MyStatName1", XblTitleManagedStatType::Number, 200 });
statList.push_back(XblTitleManagedStatistic{ "MyStatName2", XblTitleManagedStatType::String, 0, "SomeValue" });

auto asyncBlock = std::make_unique<XAsyncBlock>();
asyncBlock->queue = queue;
asyncBlock->context = nullptr;
asyncBlock->callback = [](XAsyncBlock* asyncBlock)
{
    std::unique_ptr<XAsyncBlock> asyncBlockPtr{ asyncBlock }; // Take over ownership of the XAsyncBlock*
    HRESULT hr = XAsyncGetStatus(asyncBlock, false);
};

HRESULT hr = XblTitleManagedStatsWriteAsync(
    xboxLiveContext, 
    xboxUserId, 
    statList.data(), statList.size(), 
    asyncBlock.get());
if (SUCCEEDED(hr))
{
    // The call succeeded, so release the std::unique_ptr ownership of XAsyncBlock* since the callback will take over ownership.
    // If the call fails, the std::unique_ptr will keep ownership and delete the XAsyncBlock*
    asyncBlock.release();
}
```
