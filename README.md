# Roblox Reverse-Engineering Guide for IDA

## First steps

### Decrypting Roblox

This is self-explanatory; just dump Roblox with a bypassed Scylla or decrypt it(the keyword here is ChaCha20).<br>
This step can be skipped if you're disassembling a mobile version.

### Disassembling Roblox

Bro, just Google IDA Crack and go on some russian forum!1!1!11

## Finding Important Addresses

### Finding lua_newstate
1. Open Strings view (Shift + F12)
2. Search for: `Failed to create Lua state`
3. Look at the only cross-reference (Ctrl + X or X)
4. Decompile the results (TAB or F5)
5. You should see code like this:
```c
v8 = sub_52A929C(&loc_2CC93A4, 0LL);
if ( !v8 )
    sub_62BAA48("Failed to create Lua state");
```
The function taking two parameters (second being 0) and returning a value that's checked for NULL is `lua_newstate` (sub_52A929C in this example).

### Finding luau_load
1. Open Strings view (Shift + F12)
2. Search for: `%s%.*s`
3. Look at any of the two cross-references (Ctrl + X or X)
4. Decompile the results (TAB or F5)
5. This should be in a sub that looks like this:
```c
__int64 __fastcall sub_52BA4DC(__int64 a1, const char *a2, const char *a3, int a4, unsigned int a5)
```
That sub is `luau_load` (sub_52BA4DC in this example).

### Finding lua_pushstring & lua_pushcclosurek
1. Open Strings view (Shift + F12)
2. Search for: `__index`
3. Look at the first cross-reference (Ctrl + X or X)
4. Decompile the results (TAB or F5)
5. You should see code like this:
```c
if ( *a4 )
{
  sub_5297B8C(a1, "__index");
  sub_5297DD4(a1, *a4, 0LL, 0, 0LL);
  sub_52986CC(a1, 4294967293LL);
}
```
The function taking two parameters (second being "__index") is `lua_pushstring` (sub_5297B8C in this example).

The function taking five parameters (last three being 0) is `lua_pushcclosurek` (sub_5297DD4 in this example).

### Finding lua_setfield, _G & LUA_GLOBALSINDEX
1. Open Strings view (Shift + F12)
2. Search for: `_VERSION`
3. Look at any of the two cross-references (Ctrl + X or X)
4. Decompile the results (TAB or F5)
5. You should see a sub like this:
```c
__int64 __fastcall sub_529ADA4(__int64 a1)
{
  sub_5296C98(a1, 4294957294LL);
  sub_5298750(a1, 4294957294LL, "_G");
  sub_529A300(a1, (__int64)"_G", off_680A280);
  sub_5297B00(a1, "Luau", 4LL);
  sub_5298750(a1, 4294957294LL, "_VERSION");
  sub_5297DD4(a1, (__int64)sub_529AD34, 0LL, 0, 0LL);
  sub_5297DD4(a1, (__int64)sub_529AF30, (__int64)"ipairs", 1u, 0LL);
  sub_5298750(a1, 4294967294LL, "ipairs");
  sub_5297DD4(a1, (__int64)sub_529ACDC, 0LL, 0, 0LL);
  sub_5297DD4(a1, (__int64)sub_529AF80, (__int64)"pairs", 1u, 0LL);
  sub_5298750(a1, 4294967294LL, "pairs");
  sub_5297DD4(a1, (__int64)sub_529AFCC, (__int64)"pcall", 0, (__int64)sub_529B0A8);
  sub_5298750(a1, 4294967294LL, "pcall");
  sub_5297DD4(a1, (__int64)sub_529B114, (__int64)"xpcall", 0, (__int64)sub_529B228);
  sub_5298750(a1, 4294967294LL, "xpcall");
  return 1LL;
}
```
The function taking three parameters (second being the index and third being the name of the field) is `lua_setfield` (sub_5298750 in this example).

The number 4294967294 (0xFFFFFFFE) is `_G` (-2), which refers to the second element from the top of the stack.

The number 4294957294 (0xFFFFFFF6) is `LUA_GLOBALSINDEX` (-10002), which is a pseudo-index that always refers to the global table.

### Finding luaL_register
1. Open Strings view (Shift + F12)
2. Search for: `_LOADED`
3. Look at any of the two cross-references (Ctrl + X or X)
4. Decompile the results (TAB or F5)
5. This should be in a sub that looks like this:
```c
void __fastcall sub_529A300(__int64 result, const char *a2, __int64 *a3)
```
That sub is `luaL_register` (sub_529A300 in this example).

## Quick Reference

### Essential IDA Shortcuts
- Open Strings view: `Shift + F12`
- View cross-references(xrefs): `Ctrl + X` or `X`
- Decompile: `TAB` or `F5`

### About String Search
- "Strings" view shows all strings in the binary
- Use the filter box to search
- Double-click strings to go to their location

### About Cross-references (Xrefs)
- Shows all references to the selected item
- Access via `Ctrl + X` or `X` menu
- Click references to navigate to them

### What can I do now?
As of writing this, only newstate, push & field stuff was included. This is what you can do with the stuff you got earlier:
```cpp
l = lua_newstate(&loc_2CC93A4, 0); 
lua_pushcclosurek(l, func, 0, 0, 0);
lua_setfield(l, _G, "name");
```
You can replace `func` and `name` with whatever you wish to!

In the future when I update this, you can make a script executor so you can actually use these functions!

## Notes

### Version Differences
You might see:
- Different variable names (v1, v2, etc.)
- Different parameter types (__int64, long long, etc.)
- Different function names
- Updated method patterns

Focus on the structure of the code rather than exact names.

### Found Something Different?
If you find major changes:
1. Document what's different
2. Submit a pull request with updated examples

### Need Help?
- Look for the error message strings
- Check number of parameters
- Look for NULL checks after calls
- Compare overall patterns
