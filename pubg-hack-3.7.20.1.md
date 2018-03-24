---
title: PUBG External Support for 3.7.20.1
date: 2018-3-24
tags: "hack"
categories: "hack"
---


## UWorld:


```cpp
pUWorld = mem->Read<DWORD_PTR>(mem->Read<DWORD_PTR>(i64GameBase + 0x3EB9580));
```


## GNames:


```cpp
GNames = mem->Read<DWORD_PTR>(i64GameBase + 0x3F28888);
```


## Actors:


```cpp
typedef __int64(__fastcall *DecryptF)(DWORD_PTR);
ULONGLONG globalCryptTable = 0x3C34120;
size_t calcFuncLengthEx(uint64_t funcAddress, BYTE end = 0xC3)
{
  size_t lenght = 0;
  while (mem->Read<BYTE>(funcAddress++) != end)
    lenght++;
  return lenght;
}
template<class T> T __ROL__(T value, int count)
{
  const uint nbits = sizeof(T) * 8;
 
  if (count > 0)
  {
    count %= nbits;
    T high = value >> (nbits - count);
    if (T(-1) < 0) // signed value
      high &= ~((T(-1) << count));
    value <<= count;
    value |= high;
  }
  else
  {
    count = -count % nbits;
    T low = value << (nbits - count);
    value >>= count;
    value |= low;
  }
  return value;
}
 
inline uint8  __ROR1__(uint8  value, int count) { return __ROL__((uint8)value, -count); }
inline uint16 __ROL2__(uint16 value, int count) { return __ROL__((uint16)value, count); }
inline uint16 __ROR2__(uint16 value, int count) { return __ROL__((uint16)value, -count); }
inline uint64 __ROR8__(uint64 value, int count) { return __ROL__((uint64)value, -count); }
 
#define BYTEn(x, n)   (*((_BYTE*)&(x)+n))
#define WORDn(x, n)   (*((_WORD*)&(x)+n))
#define WORD1(x)   WORDn(x,  1)
#define BYTE1(x)   BYTEn(x,  1)   
 
PVOID codeMemAct = NULL;
DWORD_PTR DecryptActors(DWORD_PTR cryptedOffset)
{
    if (codeMemAct == NULL)
    codeMemAct = VirtualAlloc(0, 1024, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
 
  uint32 v1 = mem->Read<uint32>(cryptedOffset);
  DWORD_PTR v2 = mem->Read<DWORD_PTR>(cryptedOffset + 8);
 
  DWORD_PTR v4 = globalCryptTable + i64GameBase;
 
  DWORD_PTR targetFunc = mem->Read<DWORD_PTR>(v4 + (8 * (((unsigned __int8)(((v1 + 11) ^ (~((~BYTE2(v1) + 63) ^ 0xC1) + 69)) - 29) ^ ((unsigned __int8)__ROL1__((unsigned __int16)((v1 + 11) ^ (~((~WORD1(v1) + 63) ^ 0xFFC1) + 1861)) >> 8, 19) + 54)) % 128)));
 
  ULONG delta = mem->Read<ULONG>(targetFunc + 10);
 
  if (!mem->ReadStr(targetFunc, PVOID((__int64)codeMemAct), 9))
    return 0;
 
  auto funcL = calcFuncLengthEx((uint64_t)(targetFunc + 14 + delta));
 
  if (!mem->ReadStr(targetFunc + 14 + delta, PVOID((__int64)codeMemAct + 9), funcL))
    return 0;
 
  if (!mem->ReadStr(targetFunc + 9 + 5, PVOID((__int64)codeMemAct + 9 + funcL), 0x45))
    return 0;
 
  DecryptF Decrypt = (DecryptF)((__int64)codeMemAct);
  auto res = __ROR8__(Decrypt(~(~v2 ^ v1)), 49);
 
  ZeroMemory(codeMemAct, 1024);
 
  return res;
}
```


## Usage:


```cpp
pEntityList = DecryptActors(gll::pULevel + 0xA0);
auto ObjectCount = mem->Read<int>(pEntityList + 0x8);
auto pList = mem->Read<DWORD_PTR>(pEntityList);
for (auto nIndex = 0; nIndex < ObjectCount; nIndex++)
 AActor* Actor = mem->Read<AActor*>(pList + (nIndex * sizeof(DWORD_PTR)));
```


---


provided by @DirtyFrank