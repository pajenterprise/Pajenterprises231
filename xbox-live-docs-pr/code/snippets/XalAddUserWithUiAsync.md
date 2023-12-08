```cpp
    XAsyncBlock* asyncBlock = new XAsyncBlock();
    asyncBlock->queue = GlobalState()->queue;
    asyncBlock->context = myContext;
    asyncBlock->callback = [](XAsyncBlock* asyncBlock)
    {
        xal_user_handle_t newUser = nullptr;
        HRESULT hr = XalAddUserWithUiResult(asyncBlock, &newUser);
        if (SUCCEEDED(hr))
        {
            // Store and use newUser
        }
        else
        {
            // Display error message
        }

        delete asyncBlock;
    };

    HRESULT hr = XalAddUserWithUiAsync(nullptr, asyncBlock);
    if (FAILED(hr))
    {
        delete asyncBlock;
    }
```
