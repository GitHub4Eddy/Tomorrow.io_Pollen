-- QuickApp POLLEN (Tomorrow.io)

-- This QuickApp gives access to real-time pollen count and risk category for grass pollen, weed pollen and tree pollen of any location from Tomorrow.io

-- Pollen is a fine powder produced by trees and plants
-- Pollen can severely affect people, especially those with different ailments such as asthma and respiratory issues.

-- Grass Pollen: Pollen grains from grasses
-- Weed Pollen: Weeds are annual and perennial herbs and shrubs. A single plant may produce about a billion grains of pollen per season. 
-- Tree Pollen: Pollen from trees such as Birch and Oak

-- Risk:
-- 0: None
-- 1: Very low
-- 2: Low
-- 3: Medium
-- 4: High
-- 5: Very High


-- IMPORTANT
-- You need an API key from https://app.tomorrow.io
-- The API is free up to 500 calls / per day, 25 calls / per hour, 3 calls / per second (Pollen is 5% of rate limit)
-- You need to create your location in https://app.tomorrow.io/locations to get a location ID

-- ToDo:
-- Automatic retrieval of location ID from Tomorrow.io
-- Further testing of measurements


-- Version 0.1 (15th August 2021)
-- Initial working version


-- Variables (mandatory): 
-- apiKey = Get your free API key from Tomorrow.io
-- location =  Tomorrow.io location ID (created in https://app.tomorrow.io/locations)
-- interval = [number] in seconds time to get the data from the API
-- timeout = [number] in seconds for http timeout
-- debugLevel = Number (1=some, 2=few, 3=all, 4=simulation mode) (default = 1)
-- icon = [numbber] User defined icon number (add the icon via an other device and lookup the number)


-- No editing of this code is needed 


class 'Grass'(QuickAppChild)
function Grass:__init(dev)
  QuickAppChild.__init(self,dev)
  --self:trace("Grass initiated, deviceId:",self.id)
end
function Grass:updateValue(data) 
  self:updateProperty("value",tonumber(data.ValueGrass))
  self:updateProperty("unit", "")
  self:updateProperty("log", data.RiskGrass)
end

class 'Weed'(QuickAppChild)
function Weed:__init(dev)
  QuickAppChild.__init(self,dev)
  --self:trace("Weed initiated, deviceId:",self.id)
end
function Weed:updateValue(data) 
  self:updateProperty("value",tonumber(data.ValueWeed)) 
  self:updateProperty("unit", "")
  self:updateProperty("log", data.RiskWeed)
end

class 'Tree'(QuickAppChild)
function Tree:__init(dev)
  QuickAppChild.__init(self,dev)
  --self:trace("Tree initiated, deviceId:",self.id)
end
function Tree:updateValue(data) 
  self:updateProperty("value",tonumber(data.ValueTree)) 
  self:updateProperty("unit", "")
  self:updateProperty("log", data.RiskTree)
end


-- QuickApp functions


local function getChildVariable(child,varName)
  for _,v in ipairs(child.properties.quickAppVariables or {}) do
    if v.name==varName then return v.value end
  end
  return ""
end


function QuickApp:logging(level,text) -- Logging function for debug
  if tonumber(debugLevel) >= tonumber(level) then 
      self:debug(text)
  end
end


function QuickApp:updateProperties() --Update properties
  self:logging(3,"updateProperties")
  self:updateProperty("log", data.startTime)
end


function QuickApp:updateLabels() -- Update labels
  self:logging(3,"updateLabels")
  local labelText = ""
  if debugLevel == 4 then
    labelText = labelText .."SIMULATION MODE" .."\n\n"
  end
  labelText = labelText .."Grass Pollen: " ..data.ValueGrass .." (" ..data.RiskGrass ..")" .."\n"
  labelText = labelText .."Weed Pollen: " ..data.ValueWeed .." (" ..data.RiskWeed ..")" .."\n"
  labelText = labelText .."Tree Pollen:  " ..data.ValueTree .." (" ..data.RiskTree ..")" .."\n\n"

  labelText = labelText .."Measured: " ..data.startTime .."\n"

  self:logging(2,"labelText: " ..labelText)
  self:updateView("label1", "text", labelText) 
end


function QuickApp:setRisk(val) -- Set the risk indication
  self:logging(3,"setRisk")
  
  if val == 0 then
    return "No risk"
  elseif val == 1 then
    return "Very low risk"
  elseif val == 2 then
    return "Low risk"
  elseif val == 3 then
    return "Medium risk"
  elseif val == 4 then
    return "High risk"
  elseif val == 5 then
    return "Very high risk"
  else
    return ""
  end
end


function QuickApp:getValues() -- Get the values
  self:logging(3,"getValues")
  
  data.startTime = jsonTable.data.timelines[1].startTime
  data.ValueGrass = jsonTable.data.timelines[1].intervals[1].values.grassIndex
  data.ValueWeed = jsonTable.data.timelines[1].intervals[1].values.weedIndex
  data.ValueTree = jsonTable.data.timelines[1].intervals[1].values.treeIndex
  
  data.RiskGrass = self:setRisk(tonumber(data.ValueGrass))
  data.RiskWeed = self:setRisk(tonumber(data.ValueWeed))
  data.RiskTree = self:setRisk(tonumber(data.ValueTree))

  -- Check timezone and daylight saving time
  local timezone = os.difftime(os.time(), os.time(os.date("!*t",os.time())))/3600
  if os.date("*t").isdst then -- Check daylight saving time 
    timezone = timezone + 1
  end
  self:logging(3,"Timezone + dst: " ..timezone)

  -- Convert time of measurement to local timezone
  local pattern = "(%d+)-(%d+)-(%d+) (%d+):(%d+):(%d+)"
  data.startTime = data.startTime:gsub("%.000Z", "") -- Clean up date/time
  data.startTime = data.startTime:gsub("%T", " ") -- Clean up date/time
  local runyear, runmonth, runday, runhour, runminute, runseconds = data.startTime:match(pattern)
  local convertedTimestamp = os.time({year = runyear, month = runmonth, day = runday, hour = runhour, min = runminute, sec = runseconds})
  data.startTime = os.date("%d-%m-%Y %X", convertedTimestamp + (timezone*3600))
end


function QuickApp:checkLocation() -- Get the location ID from Tomorrow.io
  self:logging(3,"Start checkLocation")
  
  local url = "https://api.tomorrow.io/v4/locations?apikey=" ..apiKey
  self:logging(2,"URL: " ..url)

  http:request(url, {
    options = {data = Method, method = "GET", headers = {["Content-Type"] = "application/json",["Accept"] = "application/json",}},
    
      success = function(response)
        self:logging(3,"response status: " ..response.status)
        self:logging(3,"headers: " ..response.headers["Content-Type"])
        self:logging(2,"Response data: " ..response.data)

        if response.data == nil or response.data == "" or response.data == "[]" or response.status > 200 then -- Check for empty result
          self:warning("Temporarily no location from Tomorrow.io, setting default location")
          self:warning(response.data)
          location = "61137ec8533a7e0008b53f4d" -- Default location "De Bilt"
          return
        end

        local geoTable = json.decode(response.data) -- JSON decode from api to lua-table
        location = tostring(geoTable.data.locations[1].id) -- Get the FIRST location ID from Tomorrow.io
        self:logging(3, "location: " ..location)
      
      end,
      error = function(error)
        self:error('error: ' ..json.encode(error))
        self:updateProperty("log", "error: " ..json.encode(error))
      end
    }) 
  self:logging(3,"location: " ..location)
  self:setVariable("location", location)
  self:trace("Added QuickApp variable location: " ..location)
end


function QuickApp:simData() -- Simulate Tomorrow.io API
  self:logging(3,"Simulation mode")
  local apiResult = '{"data":{"timelines":[{"timestep":"current","startTime":"2021-08-11T07:08:00Z","endTime":"2021-08-11T07:08:00Z","intervals":[{"startTime":"2021-08-11T07:08:00Z","values":{"weedIndex":1,"treeIndex":2,"grassIndex":3}}]}]}}'
  
  self:logging(3,"apiResult: " ..apiResult)

  jsonTable = json.decode(apiResult) -- Decode the json string from api to lua-table 
  
  self:getValues()
  self:updateLabels()
  self:updateProperties()

  for id,child in pairs(self.childDevices) do 
    child:updateValue(data) 
  end
  
  self:logging(3,"SetTimeout " ..interval .." seconds")
  fibaro.setTimeout(interval*1000, function() 
     self:simData()
  end)
end


function QuickApp:getData()
  self:logging(3,"Start getData")
  
  --if location == "" or location == "0" or location == nil then 
    --self:checkLocation() -- Check your location ID from Tomorrow.io
  --end
  
  self:logging(2,"URL: " ..address)
    
  http:request(address, {
    options = {data = Method, method = "GET", headers = {["Content-Type"] = "application/json",["Accept"] = "application/json",}},
    
      success = function(response)
        self:logging(3,"response status: " ..response.status)
        self:logging(3,"headers: " ..response.headers["Content-Type"])
        self:logging(2,"Response data: " ..response.data)

        if response.data == nil or response.data == "" or response.data == "[]" or response.status > 200 then -- Check for empty result
          self:warning("Temporarily no data from Tomorrow.io")
          self:warning(response.data)
          return
        end

        jsonTable = json.decode(response.data) -- JSON decode from api to lua-table

        self:getValues()
        self:updateLabels()
        self:updateProperties()

        for id,child in pairs(self.childDevices) do 
          child:updateValue(data) 
        end
      
      end,
      error = function(error)
        self:error('error: ' ..json.encode(error))
        self:updateProperty("log", "error: " ..json.encode(error))
      end
    }) 

  self:logging(3,"SetTimeout " ..interval .." seconds")
  fibaro.setTimeout((interval)*1000, function() 
     self:getData()
  end)
end


function QuickApp:createVariables() -- Get all Quickapp Variables or create them
  data = {}
  data.startTime = ""
  data.ValueGrass = "0"
  data.ValueWeed = "0"
  data.ValueTree = "0"
  data.RiskGrass = ""
  data.RiskWeed = ""
  data.RiskTree = ""
end


function QuickApp:getQuickAppVariables() -- Get all variables 
  apiKey = self:getVariable("apiKey")
  location = self:getVariable("location")
  interval = tonumber(self:getVariable("interval")) 
  httpTimeout = tonumber(self:getVariable("httpTimeout")) 
  debugLevel = tonumber(self:getVariable("debugLevel"))
  local icon = tonumber(self:getVariable("icon")) 

  if apiKey == "" or apiKey == "0" or apiKey == nil then
    apiKey = "0" 
    self:setVariable("apiKey",apiKey)
    self:trace("Added QuickApp variable apiKey")
  end
  if location == "" or location == "0" or location == nil then
    location = "0" 
    self:setVariable("location",location)
    self:trace("Added QuickApp variable location")
  end
  if interval == "" or interval == nil then
    interval = "3600" -- (every 60 minutes)
    self:setVariable("interval",interval)
    self:trace("Added QuickApp variable interval: " ..interval)
    interval = tonumber(interval)
  end  
  if httpTimeout == "" or httpTimeout == nil then
    httpTimeout = "5" -- timeoout in seconds
    self:setVariable("httpTimeout",httpTimeout)
    self:trace("Added QuickApp variable httpTimeout")
    httpTimeout = tonumber(httpTimeout)
  end
  if debugLevel == "" or debugLevel == nil then
    debugLevel = "1" -- Default value for debugLevel response in seconds
    self:setVariable("debugLevel",debugLevel)
    self:trace("Added QuickApp variable debugLevel")
    debugLevel = tonumber(debugLevel)
  end
  if icon == "" or icon == nil then 
    icon = "0" -- Default icon
    self:setVariable("icon",icon)
    self:trace("Added QuickApp variable icon")
    icon = tonumber(icon)
  end
  if icon ~= 0 then 
    self:updateProperty("deviceIcon", icon) -- set user defined icon 
  end
  
  address = "https://api.tomorrow.io/v4/timelines?location=" ..location .."&fields=weedIndex&fields=treeIndex&fields=grassIndex&units=metric&timesteps=current&apikey=" ..apiKey

  if apiKey == nil or apiKey == ""  or apiKey == "0" then -- Check mandatory API key 
    self:error("API key is empty! Get your free API key from Tomorrow.io")
    self:warning("No API Key: Switched to Simulation Mode")
    debugLevel = 4 -- Simulation mode due to empty API key
  end
  
  if location == nil or location == ""  or location == "0" then -- Check mandatory location   
    self:error("Location ID is empty! Create your location at your account on Tomorrow.io and copy the location ID to the quickapp variable")
    self:warning("No location: Switched to Simulation Mode")
    debugLevel = 4 -- Simulation mode due to empty location 
  end

end


function QuickApp:setupChildDevices()
  local cdevs = api.get("/devices?parentId="..self.id) or {} -- Pick up all my children 
  function self:initChildDevices() end -- Null function, else Fibaro calls it after onInit()...

  if #cdevs==0 then -- No children, create children
    local initChildData = { 
      {className="Grass", name="Grass", type="com.fibaro.multilevelSensor", value=0},
      {className="Weed", name="Weed", type="com.fibaro.multilevelSensor", value=0},
      {className="Tree", name="Tree", type="com.fibaro.multilevelSensor", value=0},
    }
    for _,c in ipairs(initChildData) do
      local child = self:createChildDevice(
        {name = c.name,
          type=c.type,
          value=c.value,
          unit=c.unit,
          initialInterfaces = {}, 
        },
        _G[c.className] -- Fetch class constructor from class name
      )
      child:setVariable("className",c.className)  -- Save class name so we know when we load it next time
    end   
  else 
    for _,child in ipairs(cdevs) do
      local className = getChildVariable(child,"className") -- Fetch child class name
      local childObject = _G[className](child) -- Create child object from the constructor name
      self.childDevices[child.id]=childObject
      childObject.parent = self -- Setup parent link to device controller 
    end
  end
end


function QuickApp:onInit()
  __TAG = fibaro.getName(plugin.mainDeviceId) .." ID:" ..plugin.mainDeviceId
  self:debug("onInit") 
  
  self:setupChildDevices() -- Setup the Child Devices
  self:getQuickAppVariables() -- Get Quickapp Variables or create them
  self:createVariables() -- Create Variables
  
  http = net.HTTPClient({timeout=httpTimeout*1000})

  if tonumber(debugLevel) >= 4 then 
    self:simData() -- Go in simulation
  else
    self:getData() -- Get data from API
  end
end

--EOF
