# Roblox Reverse-Engineering Guide for IDA

## First steps

### Decrypting Roblox

This is self-explanatory; just dump Roblox with a bypassed Scylla or decrypt it(the keyword here is ChaCha20).

### Disassembling Roblox

Bro, just Google IDA Crack and go on some russian forum!1!1!11

### Rebasing

This is optional, but the examples we have are rebased to 0x40000.

To rebase go to: "Edit/Segments/Rebase Program".<br>
Only change the hex address that's already there, nothing else.

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

### Finding lua_pushstring
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

### Finding lua_pushcclosurek
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
The function taking five parameters (last three being 0) is `lua_pushcclosurek` (sub_5297DD4 in this example).

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
