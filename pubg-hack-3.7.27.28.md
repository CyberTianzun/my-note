---
title: PUBG External Support for 3.7.27.28
date: 2018-4-8
tags: "hack"
categories: "hack"
---


## code



```cpp
// exports

#define TABLE 0x3c71120

struct uint128_t {
  uint64_t low;
  uint64_t high;
};

uint64_t tsl_decrypt_world(struct tsl *tsl, uint64_t world) {
  struct uint128_t xmm;
  if (!READ(world, &xmm, 16)) {
    return 0;
  }
  uint32_t key = (uint32_t)xmm.low;
  uint16_t x;
  uint16_t y;
  x = ror2(key + 85, -85);
  y = x ^ (ror2(IDA_HIWORD(key) - 95, 95) + 55643);
  uint64_t func = READ64(GET_ADDR(TABLE) + 0x8 * ((ror1((x ^ (ror2(IDA_HIWORD(key) - 95, 95) + 91)) + 125, -125) ^ (ror1(BYTE1(y), 77) + 138)) % 128));
  if (!make_decrypt_func(tsl, func)) {
    return 0;
  }
  uint64_t ret = tsl->func(key ^ rol8(xmm.high - key, key & 7));
  memset(tsl->func, 0, 0x400);
  return ror8(ret, -17);
}

uint64_t tsl_decrypt_actor(struct tsl *tsl, uint64_t actor) {
  struct uint128_t xmm;
  if (!READ(actor, &xmm, 16)) {
    return 0;
  }
  uint32_t key = (uint32_t)xmm.low;
  uint64_t func = READ64(GET_ADDR(TABLE) + 0x8 * ((rol1(((BYTE2(key) + 84) ^ rol2(key + 102, 102)) - 106, -106) ^ (ror1(((uint16_t)((WORD1(key) - 114 + 25286) ^ rol2(key + 102, 102)) >> 8) - 10, 10) + 244)) % 128));
  if (!make_decrypt_func(tsl, func)) {
    return 0;
  }
  uint64_t ret = tsl->func(ror8(xmm.high, key & 7) + key);
  memset(tsl->func, 0, 0x400);
  return ror8(ret, -82);
}

uint64_t tsl_decrypt_prop(struct tsl *tsl, uint64_t prop) {
  struct uint128_t xmm;
  if (!READ(prop, &xmm, 16)) {
    return 0;
  }
  uint32_t key = (uint32_t)xmm.low;
  uint16_t x;
  uint16_t y;
  uint16_t z;
  uint16_t w;
  uint8_t q;
  uint8_t e;
  uint32_t r; // uint8_t
  uint64_t t;
  if (key & 1) {
    x = rol2(key, 31);
  }
  else {
    x = ror2(key, 31);
  }
  y = key >> 16;
  if (key & 0x10000) {
    z = rol2(y, -125);
  }
  else {
    z = ror2(y, -125);
  }
  w = x ^ (z + 54543);
  q = x ^ (z + 15);
  if (w & 1) {
    e = rol1(q, -105);
  }
  else {
    e = ror1(q, -105);
  }
  r = e ^ ((uint8_t)~(~BYTE1(w) + 7) + 34);
  if (key & 2) {
    t = xmm.high - key;
  }
  else {
    t = key + xmm.high;
  }
  uint64_t func = READ64(GET_ADDR(TABLE) + 0x8 * (r % 128));
  if (!make_decrypt_func(tsl, func)) {
    return 0;
  }
  uint64_t ret = tsl->func(~t);
  memset(tsl->func, 0, 0x400);
  return ror8(ret, -45);
}
```


## Usage:


```cpp

// 3.7.27.28
 
struct tsl tsl;

if (!tsl_init(&tsl)) {
  // what?
}

uint64_t uworld = tsl_decrypt_world(&tsl, g_base_addr + 0x41cdb30);
uint64_t level = tsl_decrypt_prop(&tsl, uworld + 0x30);
uint64_t game_inst = READ64(uworld + 0x148);
uint64_t local_player = tsl_decrypt_prop(&tsl, READ64(game_inst + 0x38));
uint64_t player_controller = tsl_decrypt_prop(&tsl, local_player + 0x30);
uint64_t player_camera_manager = READ64(player_controller + 0x498);
uint64_t viewport_client = READ64(local_player + 0x60);
uint64_t pworld = READ64(viewport_client + 0x80);

uint64_t actor = tsl_decrypt_actor(&tsl, level + 0xa0);
uint64_t actor_list = READ64(actor);
uint32_t actor_count = READ32(actor + 0x8);

uint64_t local_player_actor = tsl_decrypt_prop(&tsl, player_controller + 0x470);

uint64_t gnames = READ64(g_base_addr + 0x3f66a28);

tsl_finit(&tsl);
```


---


provided by @modnation
