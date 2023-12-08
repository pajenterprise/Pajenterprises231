```cpp
    // Call XalUserGetId here to ensure all vetos (gametag banned, etc) have passed
    uint64_t xuid = 0;
    HRESULT hr = XalUserGetId(newUser, &xuid);

    if (SUCCEEDED(hr))
    {
        XblContextHandle newXblContext = nullptr;
        hr = XblContextCreateHandle(newUser, &newXblContext);

        if (SUCCEEDED(hr))
        {
            // TODO: Close the previous XblContext, if one existed
            // TODO: Set the current XblContext to be the newXblContext
        }
        else
        {
            // LOG: Failed to create XblContextHandle
        }
    }
    else
    {
        // LOG: Failed to get user id
    }
```
