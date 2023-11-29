local _decompile = assert(decompile or syn_decompile)
local _getscriptclosure = assert(getscriptclosure)
local _getrenv = assert(getrenv or syn_getrenv)
local _getgenv = assert(getgenv or syn_getgenv)
local _getsenv = assert(getsenv or syn_getsenv)
local _getmenv = assert(getmenv or syn_getmenv)
local _getprotos = assert(getprotos or debug.getprotos)
local _getconstants = assert(getconstants or debug.getconstants)
local _getinfo = debug.info
local stringsplit = string.split
local stringmatch = string.match
local stringfind = string.find
local stringgmatch = string.gmatch
local stringgsub = string.gsub
local tableclear = table.clear
local tableinsert = table.insert
local tableremove = table.remove

local IsA = game.IsA

local GetFuncs
do
local Funcs = {}
function GetFuncs(Closure, Globals)
local Protos = _getprotos(Closure)
if Protos and #Protos > 0 then
for Index, Proto in ipairs(_getprotos(Closure)) do
local Name, Args, VarArg = _getinfo(Proto, "na")
local Protos = _getprotos(Proto)
if #Name > 0 and not Globals[Name] then -- we dont need globals
tableinsert(Funcs, {Name, Args, VarArg, _getconstants(Proto)}) 
end
if Protos and #Protos > 0 then
GetFuncs(Proto, Globals)
end
end
end
return Funcs
end
end

_getgenv().decompile = (function(Script, ...)
if typeof(Script) == "Instance" then
local isModuleScript = IsA(Script, "ModuleScript")
if isModuleScript or (IsA(Script, "LocalScript") and not Script.Disabled) then
local Success, Globals = pcall(((isModuleScript and require) or _getsenv), Script)
local Closure = _getscriptclosure(Script)
if isModuleScript and Success then
if type(Globals) ~= "table" then
Success, Globals = pcall(_getmenv, Script)
end
end
if Success and Closure then
local Source = _decompile(Script, ...)
if Source then
do -- local function names
local Funcs = GetFuncs(Closure, Globals)
for Match in stringgmatch(Source, "function %w+%.%w+%b()") do
local Name = stringmatch(Match, "%.(%w+)")
for Iteration = 1, #Funcs do
if Funcs[Iteration][1] == Name then
tableremove(Funcs, Iteration)
break
end
end
end
for Proto in stringgmatch(Source, "%s+local function %l%d+%b()") do
local Spaces = stringmatch(Proto, "%s+")
local GeneratedName = stringmatch(Proto, "%l%d+")
local ArgPattern = stringmatch(Proto, "%b()")
local _, Args = stringgsub(ArgPattern, "p%d+", "")
local IsVarArg = stringfind(ArgPattern, "...", 1, true) ~= nil
local ProtoClosure = stringmatch(Source, "local function " .. GeneratedName .. "%b().+" .. Spaces .. "end;") or ""
ProtoClosure = stringgsub(ProtoClosure, "local function %l%d+%b().+end;$", function(Match)
local Split = stringsplit(Match, Spaces .. "end;\n")
local NumSplit = #Split
if NumSplit > 1 then
for Index = 1, NumSplit do
local SplitI = Split[Index]
if stringmatch(SplitI, GeneratedName) then
return SplitI .. Spaces .. "end;"
end
end
end
return Match
end)
for Iteration = 1, #Funcs do
local Func = Funcs[Iteration]
local Name = Func[1]
if Args == Func[2] then
if IsVarArg == Func[3] then
local Constants = Func[4]
local HasConstants = true
for Index = 1, #Constants do
local Constant = Constants[Index]
if type(Constant) == "string" then
if not stringfind(ProtoClosure, Constant, 1, true) then
HasConstants = false
break
end
end
end
if HasConstants then
tableremove(Funcs, Iteration)
Source = stringgsub(Source, GeneratedName, Name)
break
else
continue
end
end
end
end
end
tableclear(Funcs)
end
return Source
end
end
end
end
return _decompile(Script, ...)
end)
