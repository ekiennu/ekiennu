for _,v in pairs(data) do
	for prop,val in pairs(v[3]) do
		if type(val) == "table" then
			insts[v[1]][prop] = insts[val[1]]
		else
			insts[v[1]][prop] = val
		end
	end
end

return insts[1]

local funcs = {}
funcs.Update = function(self)
	local cursorPos = self.TextBox.CursorPosition
	local text = self.TextBox.Text
	if text == "" then self.TextBox.Position = UDim2.new(0,2,0,0) return end
	if cursorPos == -1 then return end

	local cursorText = text:sub(1,cursorPos-1)
	local pos = nil
	local leftEnd = -self.TextBox.Position.X.Offset
	local rightEnd = leftEnd + self.View.AbsoluteSize.X

	local totalTextSize = TextService:GetTextSize(text,self.TextBox.TextSize,self.TextBox.Font,Vector2.new(999999999,100)).X
	local cursorTextSize = TextService:GetTextSize(cursorText,self.TextBox.TextSize,self.TextBox.Font,Vector2.new(999999999,100)).X

	if cursorTextSize > rightEnd then
		pos = math.max(-2,cursorTextSize - self.View.AbsoluteSize.X + 2)
	elseif cursorTextSize < leftEnd then
		pos = math.max(-2,cursorTextSize-2)
	elseif totalTextSize < rightEnd then
		pos = math.max(-2,totalTextSize - self.View.AbsoluteSize.X + 2)
	end

	if pos then
		self.TextBox.Position = UDim2.new(0,-pos,0,0)
		self.TextBox.Size = UDim2.new(1,pos,1,0)
	end
end

local mt = {}
mt.__index = funcs

local function convert(textbox)
	local obj = setmetatable({OffsetX = 0, TextBox = textbox},mt)

	local view = Instance.new("Frame")
	view.BackgroundTransparency = textbox.BackgroundTransparency
	view.BackgroundColor3 = textbox.BackgroundColor3
	view.BorderSizePixel = textbox.BorderSizePixel
	view.BorderColor3 = textbox.BorderColor3
	view.Position = textbox.Position
	view.Size = textbox.Size
	view.ClipsDescendants = true
	view.Name = textbox.Name
	view.ZIndex = 10
	textbox.BackgroundTransparency = 1
	textbox.Position = UDim2.new(0,4,0,0)
	textbox.Size = UDim2.new(1,-8,1,0)
	textbox.TextXAlignment = Enum.TextXAlignment.Left
	textbox.Name = "Input"
	table.insert(text1,textbox)
	table.insert(shade2,view)

	obj.View = view

	textbox.Changed:Connect(function(prop)
		if prop == "Text" or prop == "CursorPosition" or prop == "AbsoluteSize" then
			obj:Update()
		end
	end)

	obj:Update()

	view.Parent = textbox.Parent
	textbox.Parent = view

	return obj
end

return {convert = convert}

if string.find(obj.Name,' ') then
	fullname = '["'..obj.Name..'"]'
	period = false
else
	fullname = obj.Name
	period = true
end

local getS = obj
local parent = obj
local service = ''

if getS.Parent ~= game then
	repeat
		getS = getS.Parent
		service = getS.ClassName
	until getS.Parent == game
end

if parent.Parent ~= getS then
	repeat
		parent = parent.Parent
		if string.find(tostring(parent),' ') then
			if period then
				fullname = '["'..parent.Name..'"].'..fullname
			else
				fullname = '["'..parent.Name..'"]'..fullname
			end
			period = false
		else
			if period then
				fullname = parent.Name..'.'..fullname
			else
				fullname = parent.Name..''..fullname
			end
			period = true
		end
	until parent.Parent == getS
elseif string.find(tostring(parent),' ') then
	fullname = '["'..parent.Name..'"]'
	period = false
end

if period then
	return 'game:GetService("'..service..'").'..fullname
else
	return 'game:GetService("'..service..'")'..fullname
end

			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)
	gui.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
			dragInput = input
		end
	end)
	UserInputService.InputChanged:Connect(function(input)
		if input == dragInput and dragging then
			update(input)
		end
	end)
end)

local function registerEvent(name,sets)
	events[name] = {
		commands = {},
		sets = sets or {}
	}
end

local onEdited = nil

local function fireEvent(name,...)
	local args = {...}
	local event = events[name]
	if event then
		for i,cmd in pairs(event.commands) do
			local metCondition = true
			for idx,set in pairs(event.sets) do
				local argVal = args[idx]
				local cmdSet = cmd[2][idx]
				local condType = set.Type
				if condType == "Player" then
					if cmdSet == 0 then
						metCondition = metCondition and (tostring(Players.LocalPlayer) == argVal)
					elseif cmdSet ~= 1 then
						metCondition = metCondition and table.find(getPlayer(cmdSet,Players.LocalPlayer),argVal)
					end
				elseif condType == "String" then
					if cmdSet ~= 0 then
						metCondition = metCondition and string.find(argVal:lower(),cmdSet:lower())
					end
				elseif condType == "Number" then
					if cmdSet ~= 0 then
						metCondition = metCondition and tonumber(argVal)<=tonumber(cmdSet)
					end
				end
				if not metCondition then break end
			end

			if metCondition then
				pcall(task.spawn(function()
					local cmdStr = cmd[1]
					for count,arg in pairs(args) do
						cmdStr = cmdStr:gsub("%$"..count,arg)
					end
					wait(cmd[3] or 0)
					execCmd(cmdStr)
				end))
			end
		end
	end
end

local main = create({
	{1,"Frame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BackgroundTransparency=1,BorderSizePixel=0,Name="EventEditor",Position=UDim2.new(0.5,-175,0,-500),Size=UDim2.new(0,350,0,20),ZIndex=10,}},
	{2,"Frame",{BackgroundColor3=currentShade2,BorderSizePixel=0,Name="TopBar",Parent={1},Size=UDim2.new(1,0,0,20),ZIndex=10,}},
	{3,"TextLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Title",Parent={2},Position=UDim2.new(0,0,0,0),Size=UDim2.new(1,0,0.95,0),Text="Event Editor",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=Enum.TextXAlignment.Center,ZIndex=10,}},
	{4,"TextButton",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Close",Parent={2},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,ZIndex=10,}},
	{5,"ImageLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Image="rbxassetid://5054663650",Parent={4},Position=UDim2.new(0,5,0,5),Size=UDim2.new(0,10,0,10),ZIndex=10,}},
	{6,"Frame",{BackgroundColor3=currentShade1,BorderSizePixel=0,Name="Content",Parent={1},Position=UDim2.new(0,0,0,20),Size=UDim2.new(1,0,0,202),ZIndex=10,}},
	{7,"ScrollingFrame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,BottomImage="rbxasset://textures/ui/Scroll/scroll-middle.png",CanvasSize=UDim2.new(0,0,0,100),Name="List",Parent={6},Position=UDim2.new(0,5,0,5),ScrollBarImageColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),ScrollBarThickness=8,Size=UDim2.new(1,-10,1,-10),TopImage="rbxasset://textures/ui/Scroll/scroll-middle.png",ZIndex=10,}},
	{8,"Frame",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Name="Holder",Parent={7},Size=UDim2.new(1,0,1,0),ZIndex=10,}},
	{9,"UIListLayout",{Parent={8},SortOrder=2,}},
	{10,"Frame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BackgroundTransparency=1,BorderColor3=Color3.new(0.3137255012989,0.3137255012989,0.3137255012989),BorderSizePixel=0,ClipsDescendants=true,Name="Settings",Parent={6},Position=UDim2.new(1,0,0,0),Size=UDim2.new(0,150,1,0),ZIndex=10,}},
	{11,"Frame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),Name="Slider",Parent={10},Position=UDim2.new(0,-150,0,0),Size=UDim2.new(1,0,1,0),ZIndex=10,}},
	{12,"Frame",{BackgroundColor3=Color3.new(0.23529413342476,0.23529413342476,0.23529413342476),BorderColor3=Color3.new(0.3137255012989,0.3137255012989,0.3137255012989),BorderSizePixel=0,Name="Line",Parent={11},Size=UDim2.new(0,1,1,0),ZIndex=10,}},
	{13,"ScrollingFrame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,BottomImage="rbxasset://textures/ui/Scroll/scroll-middle.png",CanvasSize=UDim2.new(0,0,0,100),Name="List",Parent={11},Position=UDim2.new(0,0,0,25),ScrollBarImageColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),ScrollBarThickness=8,Size=UDim2.new(1,0,1,-25),TopImage="rbxasset://textures/ui/Scroll/scroll-middle.png",ZIndex=10,}},
	{14,"Frame",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Name="Holder",Parent={13},Size=UDim2.new(1,0,1,0),ZIndex=10,}},
	{15,"UIListLayout",{Parent={14},SortOrder=2,}},
	{16,"TextLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Title",Parent={11},Size=UDim2.new(1,0,0,20),Text="Event Settings",TextColor3=Color3.new(1,1,1),TextSize=14,ZIndex=10,}},
	{17,"TextButton",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),Font=3,Name="Close",BorderSizePixel=0,Parent={11},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),Text="<",TextColor3=Color3.new(1,1,1),TextSize=18,ZIndex=10,}},
	{18,"Folder",{Name="Templates",Parent={10},}},
	{19,"Frame",{BackgroundColor3=Color3.new(0.19607844948769,0.19607844948769,0.19607844948769),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),Name="Players",Parent={18},Position=UDim2.new(0,0,0,25),Size=UDim2.new(1,0,0,86),Visible=false,ZIndex=10,}},
	{20,"TextLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Title",Parent={19},Size=UDim2.new(1,0,0,20),Text="Choose Players",TextColor3=Color3.new(1,1,1),TextSize=14,ZIndex=10,}},
	{21,"TextLabel",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="Any",Parent={19},Position=UDim2.new(0,5,0,42),Size=UDim2.new(1,-10,0,20),Text="Any Player",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{22,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="Button",Parent={21},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{23,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={22},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{24,"TextLabel",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="Me",Parent={19},Position=UDim2.new(0,5,0,20),Size=UDim2.new(1,-10,0,20),Text="Me Only",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{25,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="Button",Parent={24},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{26,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={25},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{27,"TextBox",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,ClearTextOnFocus=false,Font=3,Name="Custom",Parent={19},PlaceholderColor3=Color3.new(0.47058826684952,0.47058826684952,0.47058826684952),PlaceholderText="Custom Player Set",Position=UDim2.new(0,5,0,64),Size=UDim2.new(1,-35,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{28,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="CustomButton",Parent={19},Position=UDim2.new(1,-25,0,64),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{29,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={28},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{30,"Frame",{BackgroundColor3=Color3.new(0.19607844948769,0.19607844948769,0.19607844948769),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),Name="Strings",Parent={18},Position=UDim2.new(0,0,0,25),Size=UDim2.new(1,0,0,64),Visible=false,ZIndex=10,}},
	{31,"TextLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Title",Parent={30},Size=UDim2.new(1,0,0,20),Text="Choose String",TextColor3=Color3.new(1,1,1),TextSize=14,ZIndex=10,}},
	{32,"TextLabel",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="Any",Parent={30},Position=UDim2.new(0,5,0,20),Size=UDim2.new(1,-10,0,20),Text="Any String",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{33,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="Button",Parent={32},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{34,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={33},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{54,"Frame",{BackgroundColor3=Color3.new(0.19607844948769,0.19607844948769,0.19607844948769),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),Name="Numbers",Parent={18},Position=UDim2.new(0,0,0,25),Size=UDim2.new(1,0,0,64),Visible=false,ZIndex=10,}},
	{55,"TextLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Title",Parent={54},Size=UDim2.new(1,0,0,20),Text="Choose String",TextColor3=Color3.new(1,1,1),TextSize=14,ZIndex=10,}},
	{56,"TextLabel",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="Any",Parent={54},Position=UDim2.new(0,5,0,20),Size=UDim2.new(1,-10,0,20),Text="Any Number",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{57,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="Button",Parent={56},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{58,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={57},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{59,"TextBox",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,ClearTextOnFocus=false,Font=3,Name="Custom",Parent={54},PlaceholderColor3=Color3.new(0.47058826684952,0.47058826684952,0.47058826684952),PlaceholderText="Number",Position=UDim2.new(0,5,0,42),Size=UDim2.new(1,-35,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{60,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="CustomButton",Parent={54},Position=UDim2.new(1,-25,0,42),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{61,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={60},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{35,"TextBox",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,ClearTextOnFocus=false,Font=3,Name="Custom",Parent={30},PlaceholderColor3=Color3.new(0.47058826684952,0.47058826684952,0.47058826684952),PlaceholderText="Match String",Position=UDim2.new(0,5,0,42),Size=UDim2.new(1,-35,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{36,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="CustomButton",Parent={30},Position=UDim2.new(1,-25,0,42),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{37,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={36},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{38,"Frame",{BackgroundColor3=Color3.new(0.19607844948769,0.19607844948769,0.19607844948769),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),Name="DelayEditor",Parent={18},Position=UDim2.new(0,0,0,25),Size=UDim2.new(1,0,0,24),Visible=false,ZIndex=10,}},
	{39,"TextBox",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,Font=3,Name="Secs",Parent={38},PlaceholderColor3=Color3.new(0.47058826684952,0.47058826684952,0.47058826684952),Position=UDim2.new(0,60,0,2),Size=UDim2.new(1,-65,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{40,"TextLabel",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="Label",Parent={39},Position=UDim2.new(0,-55,0,0),Size=UDim2.new(1,0,1,0),Text="Delay (s):",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{41,"Frame",{BackgroundColor3=currentShade1,BorderSizePixel=0,ClipsDescendants=true,Name="EventTemplate",Parent={6},Size=UDim2.new(1,0,0,20),Visible=false,ZIndex=10,}},
	{42,"TextButton",{BackgroundColor3=currentText1,BackgroundTransparency=1,Font=3,Name="Expand",Parent={41},Size=UDim2.new(0,20,0,20),Text=">",TextColor3=Color3.new(1,1,1),TextSize=18,ZIndex=10,}},
	{43,"TextLabel",{BackgroundColor3=currentText1,BackgroundTransparency=1,Font=3,Name="EventName",Parent={41},Position=UDim2.new(0,25,0,0),Size=UDim2.new(1,-25,0,20),Text="OnSpawn",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{44,"Frame",{BackgroundColor3=Color3.new(0.19607844948769,0.19607844948769,0.19607844948769),BorderSizePixel=0,BackgroundTransparency=1,ClipsDescendants=true,Name="Cmds",Parent={41},Position=UDim2.new(0,0,0,20),Size=UDim2.new(1,0,1,-20),ZIndex=10,}},
	{45,"Frame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BorderColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),Name="Add",Parent={44},Position=UDim2.new(0,0,1,-20),Size=UDim2.new(1,0,0,20),ZIndex=10,}},
	{46,"TextBox",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,ClearTextOnFocus=false,Font=3,Parent={45},PlaceholderColor3=Color3.new(0.7843137383461,0.7843137383461,0.7843137383461),PlaceholderText="Add new command",Position=UDim2.new(0,5,0,0),Size=UDim2.new(1,-10,1,0),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{47,"Frame",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Name="Holder",Parent={44},Size=UDim2.new(1,0,1,-20),ZIndex=10,}},
	{48,"UIListLayout",{Parent={47},SortOrder=2,}},
	{49,"Frame",{currentShade1,BorderSizePixel=0,ClipsDescendants=true,Name="CmdTemplate",Parent={6},Size=UDim2.new(1,0,0,20),Visible=false,ZIndex=10,}},
	{50,"TextBox",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,ClearTextOnFocus=false,Font=3,Parent={49},PlaceholderColor3=Color3.new(1,1,1),Position=UDim2.new(0,5,0,0),Size=UDim2.new(1,-45,0,20),Text="a\\b\\c\\d",TextColor3=currentText1,TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{51,"TextButton",{BackgroundColor3=currentShade1,BorderSizePixel=0,Font=3,Name="Delete",Parent={49},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),Text="X",TextColor3=Color3.new(1,1,1),TextSize=18,ZIndex=10,}},
	{52,"TextButton",{BackgroundColor3=currentShade1,BorderSizePixel=0,Font=3,Name="Settings",Parent={49},Position=UDim2.new(1,-40,0,0),Size=UDim2.new(0,20,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=18,ZIndex=10,}},
	{53,"ImageLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Image="rbxassetid://1204397029",Parent={52},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),ZIndex=10,}},
})
main.Name = randomString()
local mainFrame = main:WaitForChild("Content")
local eventList = mainFrame:WaitForChild("List")
local eventListHolder = eventList:WaitForChild("Holder")
local cmdTemplate = mainFrame:WaitForChild("CmdTemplate")
local eventTemplate = mainFrame:WaitForChild("EventTemplate")
local settingsFrame = mainFrame:WaitForChild("Settings"):WaitForChild("Slider")
local settingsTemplates = mainFrame.Settings:WaitForChild("Templates")
local settingsList = settingsFrame:WaitForChild("List"):WaitForChild("Holder")
table.insert(shade2,main.TopBar) table.insert(shade1,mainFrame) table.insert(shade2,eventTemplate)
table.insert(text1,eventTemplate.EventName) table.insert(shade1,eventTemplate.Cmds.Add) table.insert(shade1,cmdTemplate)
table.insert(text1,cmdTemplate.TextBox) table.insert(shade2,cmdTemplate.Delete) table.insert(shade2,cmdTemplate.Settings)
table.insert(scroll,mainFrame.List) table.insert(shade1,settingsFrame) table.insert(shade2,settingsFrame.Line)
table.insert(shade2,settingsFrame.Close) table.insert(scroll,settingsFrame.List) table.insert(shade2,settingsTemplates.DelayEditor.Secs)
table.insert(text1,settingsTemplates.DelayEditor.Secs) table.insert(text1,settingsTemplates.DelayEditor.Secs.Label) table.insert(text1,settingsTemplates.Players.Title)
table.insert(shade3,settingsTemplates.Players.CustomButton) table.insert(shade2,settingsTemplates.Players.Custom) table.insert(text1,settingsTemplates.Players.Custom)
table.insert(shade3,settingsTemplates.Players.Any.Button) table.insert(shade3,settingsTemplates.Players.Me.Button) table.insert(text1,settingsTemplates.Players.Any)
table.insert(text1,settingsTemplates.Players.Me) table.insert(text1,settingsTemplates.Strings.Title) table.insert(text1,settingsTemplates.Strings.Any)
table.insert(shade3,settingsTemplates.Strings.Any.Button) table.insert(shade3,settingsTemplates.Strings.CustomButton) table.insert(text1,settingsTemplates.Strings.Custom)
table.insert(shade2,settingsTemplates.Strings.Custom)
table.insert(text1,settingsTemplates.Players.Me) table.insert(text1,settingsTemplates.Numbers.Title) table.insert(text1,settingsTemplates.Numbers.Any)
table.insert(shade3,settingsTemplates.Numbers.Any.Button) table.insert(shade3,settingsTemplates.Numbers.CustomButton) table.insert(text1,settingsTemplates.Numbers.Custom)
table.insert(shade2,settingsTemplates.Numbers.Custom)

local tweenInf = TweenInfo.new(0.25,Enum.EasingStyle.Quart,Enum.EasingDirection.Out)

local currentlyEditingCmd = nil

settingsFrame:WaitForChild("Close").MouseButton1Click:Connect(function()
	settingsFrame:TweenPosition(UDim2.new(0,-150,0,0),Enum.EasingDirection.Out,Enum.EasingStyle.Quart,0.25,true)
end)

local function resizeList()
	local size = 0

	for i,v in pairs(eventListHolder:GetChildren()) do
		if v.Name == "EventTemplate" then
			size = size + 20
			if v.Expand.Rotation == 90 then
				size = size + 20*(1+(#events[v.EventName:GetAttribute("RawName")].commands or 0))
			end
		end
	end

	TweenService:Create(eventList,tweenInf,{CanvasSize = UDim2.new(0,0,0,size)}):Play()

	if size > eventList.AbsoluteSize.Y then
		eventListHolder.Size = UDim2.new(1,-8,1,0)
	else
		eventListHolder.Size = UDim2.new(1,0,1,0)
	end
end

local function resizeSettingsList()
	local size = 0

	for i,v in pairs(settingsList:GetChildren()) do
		if v:IsA("Frame") then
			size = size + v.AbsoluteSize.Y
		end
	end

	settingsList.Parent.CanvasSize = UDim2.new(0,0,0,size)

	if size > settingsList.Parent.AbsoluteSize.Y then
		settingsList.Size = UDim2.new(1,-8,1,0)
	else
		settingsList.Size = UDim2.new(1,0,1,0)
	end
end

local function setupCheckbox(button,callback)
	local enabled = button.On.BackgroundTransparency == 0

	local function update()
		button.On.BackgroundTransparency = (enabled and 0 or 1)
	end

	button.On.MouseButton1Click:Connect(function()
		enabled = not enabled
		update()
		if callback then callback(enabled) end
	end)

	return {
		Toggle = function(nocall) enabled = not enabled update() if not nocall and callback then callback(enabled) end end,
		Enable = function(nocall) if enabled then return end enabled = true update()if not nocall and callback then callback(enabled) end end,
		Disable = function(nocall) if not enabled then return end enabled = false update()if not nocall and callback then callback(enabled) end end,
		IsEnabled = function() return enabled end
	}
end

local function openSettingsEditor(event,cmd)
	currentlyEditingCmd = cmd

	for i,v in pairs(settingsList:GetChildren()) do if v:IsA("Frame") then v:Destroy() end end

	local delayEditor = settingsTemplates.DelayEditor:Clone()
	delayEditor.Secs.FocusLost:Connect(function()
		cmd[3] = tonumber(delayEditor.Secs.Text) or 0
		delayEditor.Secs.Text = cmd[3]
		if onEdited then onEdited() end
	end)
	delayEditor.Secs.Text = cmd[3]
	delayEditor.Visible = true
	table.insert(shade2,delayEditor.Secs)
	table.insert(text1,delayEditor.Secs)
	table.insert(text1,delayEditor.Secs.Label)
	delayEditor.Parent = settingsList

	for i,v in pairs(event.sets) do
		if v.Type == "Player" then
			local template = settingsTemplates.Players:Clone()
			template.Title.Text = v.Name or "Player"

			local me,any,custom

			me = setupCheckbox(template.Me.Button,function(on)
				if not on then return end
				any.Disable()
				custom.Disable()
				cmd[2][i] = 0
				if onEdited then onEdited() end
			end)

			any = setupCheckbox(template.Any.Button,function(on)
				if not on then return end
				me.Disable()
				custom.Disable()
				cmd[2][i] = 1
				if onEdited then onEdited() end
			end)

			local customTextBox = template.Custom
			custom = setupCheckbox(template.CustomButton,function(on)
				if not on then return end
				me.Disable()
				any.Disable()
				cmd[2][i] = customTextBox.Text
				if onEdited then onEdited() end
			end)

			ViewportTextBox.convert(customTextBox)
			customTextBox.FocusLost:Connect(function()
				if custom:IsEnabled() then
					cmd[2][i] = customTextBox.Text
					if onEdited then onEdited() end
				end
			end)

			local cVal = cmd[2][i]
			if cVal == 0 then
				me:Enable()
			elseif cVal == 1 then
				any:Enable()
			else
				custom:Enable()
				customTextBox.Text = cVal
			end

			template.Visible = true
			table.insert(text1,template.Title)
			table.insert(shade3,template.CustomButton)
			table.insert(shade3,template.Any.Button)
			table.insert(shade3,template.Me.Button)
			table.insert(text1,template.Any)
			table.insert(text1,template.Me)
			template.Parent = settingsList
		elseif v.Type == "String" then
			local template = settingsTemplates.Strings:Clone()
			template.Title.Text = v.Name or "String"

			local any,custom

			any = setupCheckbox(template.Any.Button,function(on)
				if not on then return end
				custom.Disable()
				cmd[2][i] = 0
				if onEdited then onEdited() end
			end)

			local customTextBox = template.Custom
			custom = setupCheckbox(template.CustomButton,function(on)
				if not on then return end
				any.Disable()
				cmd[2][i] = customTextBox.Text
				if onEdited then onEdited() end
			end)

			ViewportTextBox.convert(customTextBox)
			customTextBox.FocusLost:Connect(function()
				if custom:IsEnabled() then
					cmd[2][i] = customTextBox.Text
					if onEdited then onEdited() end
				end
			end)

			local cVal = cmd[2][i]
			if cVal == 0 then
				any:Enable()
			else
				custom:Enable()
				customTextBox.Text = cVal
			end

			template.Visible = true
			table.insert(text1,template.Title)
			table.insert(text1,template.Any)
			table.insert(shade3,template.Any.Button)
			table.insert(shade3,template.CustomButton)
			template.Parent = settingsList
		elseif v.Type == "Number" then
			local template = settingsTemplates.Numbers:Clone()
			template.Title.Text = v.Name or "Number"

			local any,custom

			any = setupCheckbox(template.Any.Button,function(on)
				if not on then return end
				custom.Disable()
				cmd[2][i] = 0
				if onEdited then onEdited() end
			end)

			local customTextBox = template.Custom
			custom = setupCheckbox(template.CustomButton,function(on)
				if not on then return end
				any.Disable()
				cmd[2][i] = customTextBox.Text
				if onEdited then onEdited() end
			end)

			ViewportTextBox.convert(customTextBox)
			customTextBox.FocusLost:Connect(function()
				cmd[2][i] = tonumber(customTextBox.Text) or 0
				customTextBox.Text = cmd[2][i]
				if custom:IsEnabled() then
					if onEdited then onEdited() end
				end
			end)

			local cVal = cmd[2][i]
			if cVal == 0 then
				any:Enable()
			else
				custom:Enable()
				customTextBox.Text = cVal
			end

			template.Visible = true
			table.insert(text1,template.Title)
			table.insert(text1,template.Any)
			table.insert(shade3,template.Any.Button)
			table.insert(shade3,template.CustomButton)
			template.Parent = settingsList
		end
	end
	resizeSettingsList()
	settingsFrame:TweenPosition(UDim2.new(0,0,0,0),Enum.EasingDirection.Out,Enum.EasingStyle.Quart,0.25,true)
end

local function defaultSettings(ev)
	local res = {}

	for i,v in pairs(ev.sets) do
		if v.Type == "Player" then
			res[#res+1] = v.Default or 0
		elseif v.Type == "String" then
			res[#res+1] = v.Default or 0
		elseif v.Type == "Number" then
			res[#res+1] = v.Default or 0
		end
	end

	return res
end

local function refreshList()
	for i,v in pairs(eventListHolder:GetChildren()) do if v:IsA("Frame") then v:Destroy() end end

	for name,event in pairs(events) do
		local eventF = eventTemplate:Clone()
		eventF.EventName.Text = name
		eventF.Visible = true
		eventF.EventName:SetAttribute("RawName", name)
		table.insert(shade2,eventF)
		table.insert(text1,eventF.EventName)
		table.insert(shade1,eventF.Cmds.Add)

		local expanded = false
		eventF.Expand.MouseButton1Down:Connect(function()
			expanded = not expanded
			eventF:TweenSize(UDim2.new(1,0,0,20 + (expanded and 20*#eventF.Cmds.Holder:GetChildren() or 0)),Enum.EasingDirection.Out,Enum.EasingStyle.Quart,0.25,true)
			eventF.Expand.Rotation = expanded and 90 or 0
			resizeList()
		end)

		local function refreshCommands()
			for i,v in pairs(eventF.Cmds.Holder:GetChildren()) do
				if v.Name == "CmdTemplate" then
					v:Destroy()
				end
			end

			eventF.EventName.Text = name..(#event.commands > 0 and " ("..#event.commands..")" or "")

			for i,cmd in pairs(event.commands) do
				local cmdF = cmdTemplate:Clone()
				local cmdTextBox = cmdF.TextBox
				ViewportTextBox.convert(cmdTextBox)
				cmdTextBox.Text = cmd[1]
				cmdF.Visible = true
				table.insert(shade1,cmdF)
				table.insert(shade2,cmdF.Delete)
				table.insert(shade2,cmdF.Settings)

				cmdTextBox.FocusLost:Connect(function()
					event.commands[i] = {cmdTextBox.Text,cmd[2],cmd[3]}
					if onEdited then onEdited() end
				end)

				cmdF.Settings.MouseButton1Click:Connect(function()
					openSettingsEditor(event,cmd)
				end)

				cmdF.Delete.MouseButton1Click:Connect(function()
					table.remove(event.commands,i)
					refreshCommands()
					resizeList()

					if currentlyEditingCmd == cmd then
						settingsFrame:TweenPosition(UDim2.new(0,-150,0,0),Enum.EasingDirection.Out,Enum.EasingStyle.Quart,0.25,true)
					end
					if onEdited then onEdited() end
				end)

				cmdF.Parent = eventF.Cmds.Holder
			end

			eventF:TweenSize(UDim2.new(1,0,0,20 + (expanded and 20*#eventF.Cmds.Holder:GetChildren() or 0)),Enum.EasingDirection.Out,Enum.EasingStyle.Quart,0.25,true)
		end

		local newBox = eventF.Cmds.Add.TextBox
		ViewportTextBox.convert(newBox)
		newBox.FocusLost:Connect(function(enter)
			if enter then
				event.commands[#event.commands+1] = {newBox.Text,defaultSettings(event),0}
				newBox.Text = ""

				refreshCommands()
				resizeList()
				if onEdited then onEdited() end
			end
		end)

		--eventF:GetPropertyChangedSignal("AbsoluteSize"):Connect(resizeList)

		eventF.Parent = eventListHolder

		refreshCommands()
	end

	resizeList()
end

local function saveData()
	local result = {}
	for i,v in pairs(events) do
		result[i] = v.commands
	end
	return HttpService:JSONEncode(result)
end

local function loadData(str)
	local data = HttpService:JSONDecode(str)
	for i,v in pairs(data) do
		if events[i] then
			events[i].commands = v
		end
	end
end

local function addCmd(event,data)
	table.insert(events[event].commands,data)
end

local function setOnEdited(f)
	if type(f) == "function" then
		onEdited = f
	end
end

main.TopBar.Close.MouseButton1Click:Connect(function()
	main:TweenPosition(UDim2.new(0.5,-175,0,-500), "InOut", "Quart", 0.5, true, nil)
end)
dragGUI(main)
main.Parent = PARENT

return {
	RegisterEvent = registerEvent,
	FireEvent = fireEvent,
	Refresh = refreshList,
	SaveData = saveData,
	LoadData = loadData,
	AddCmd = addCmd,
	Frame = main,
	SetOnEdited = setOnEdited
}

local function registerEvent(name,sets)
	events[name] = {
		commands = {},
		sets = sets or {}
	}
end

local onEdited = nil

local function fireEvent(name,...)
	local args = {...}
	local event = events[name]
	if event then
		for i,cmd in pairs(event.commands) do
			local metCondition = true
			for idx,set in pairs(event.sets) do
				local argVal = args[idx]
				local cmdSet = cmd[2][idx]
				local condType = set.Type
				if condType == "Player" then
					if cmdSet == 0 then
						metCondition = metCondition and (tostring(Players.LocalPlayer) == argVal)
					elseif cmdSet ~= 1 then
						metCondition = metCondition and table.find(getPlayer(cmdSet,Players.LocalPlayer),argVal)
					end
				elseif condType == "String" then
					if cmdSet ~= 0 then
						metCondition = metCondition and string.find(argVal:lower(),cmdSet:lower())
					end
				elseif condType == "Number" then
					if cmdSet ~= 0 then
						metCondition = metCondition and tonumber(argVal)<=tonumber(cmdSet)
					end
				end
				if not metCondition then break end
			end

			if metCondition then
				pcall(task.spawn(function()
					local cmdStr = cmd[1]
					for count,arg in pairs(args) do
						cmdStr = cmdStr:gsub("%$"..count,arg)
					end
					wait(cmd[3] or 0)
					execCmd(cmdStr)
				end))
			end
		end
	end
end

local main = create({
	{1,"Frame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BackgroundTransparency=1,BorderSizePixel=0,Name="EventEditor",Position=UDim2.new(0.5,-175,0,-500),Size=UDim2.new(0,350,0,20),ZIndex=10,}},
	{2,"Frame",{BackgroundColor3=currentShade2,BorderSizePixel=0,Name="TopBar",Parent={1},Size=UDim2.new(1,0,0,20),ZIndex=10,}},
	{3,"TextLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Title",Parent={2},Position=UDim2.new(0,0,0,0),Size=UDim2.new(1,0,0.95,0),Text="Event Editor",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=Enum.TextXAlignment.Center,ZIndex=10,}},
	{4,"TextButton",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Close",Parent={2},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,ZIndex=10,}},
	{5,"ImageLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Image="rbxassetid://5054663650",Parent={4},Position=UDim2.new(0,5,0,5),Size=UDim2.new(0,10,0,10),ZIndex=10,}},
	{6,"Frame",{BackgroundColor3=currentShade1,BorderSizePixel=0,Name="Content",Parent={1},Position=UDim2.new(0,0,0,20),Size=UDim2.new(1,0,0,202),ZIndex=10,}},
	{7,"ScrollingFrame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,BottomImage="rbxasset://textures/ui/Scroll/scroll-middle.png",CanvasSize=UDim2.new(0,0,0,100),Name="List",Parent={6},Position=UDim2.new(0,5,0,5),ScrollBarImageColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),ScrollBarThickness=8,Size=UDim2.new(1,-10,1,-10),TopImage="rbxasset://textures/ui/Scroll/scroll-middle.png",ZIndex=10,}},
	{8,"Frame",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Name="Holder",Parent={7},Size=UDim2.new(1,0,1,0),ZIndex=10,}},
	{9,"UIListLayout",{Parent={8},SortOrder=2,}},
	{10,"Frame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BackgroundTransparency=1,BorderColor3=Color3.new(0.3137255012989,0.3137255012989,0.3137255012989),BorderSizePixel=0,ClipsDescendants=true,Name="Settings",Parent={6},Position=UDim2.new(1,0,0,0),Size=UDim2.new(0,150,1,0),ZIndex=10,}},
	{11,"Frame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),Name="Slider",Parent={10},Position=UDim2.new(0,-150,0,0),Size=UDim2.new(1,0,1,0),ZIndex=10,}},
	{12,"Frame",{BackgroundColor3=Color3.new(0.23529413342476,0.23529413342476,0.23529413342476),BorderColor3=Color3.new(0.3137255012989,0.3137255012989,0.3137255012989),BorderSizePixel=0,Name="Line",Parent={11},Size=UDim2.new(0,1,1,0),ZIndex=10,}},
	{13,"ScrollingFrame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,BottomImage="rbxasset://textures/ui/Scroll/scroll-middle.png",CanvasSize=UDim2.new(0,0,0,100),Name="List",Parent={11},Position=UDim2.new(0,0,0,25),ScrollBarImageColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),ScrollBarThickness=8,Size=UDim2.new(1,0,1,-25),TopImage="rbxasset://textures/ui/Scroll/scroll-middle.png",ZIndex=10,}},
	{14,"Frame",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Name="Holder",Parent={13},Size=UDim2.new(1,0,1,0),ZIndex=10,}},
	{15,"UIListLayout",{Parent={14},SortOrder=2,}},
	{16,"TextLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Title",Parent={11},Size=UDim2.new(1,0,0,20),Text="Event Settings",TextColor3=Color3.new(1,1,1),TextSize=14,ZIndex=10,}},
	{17,"TextButton",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),Font=3,Name="Close",BorderSizePixel=0,Parent={11},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),Text="<",TextColor3=Color3.new(1,1,1),TextSize=18,ZIndex=10,}},
	{18,"Folder",{Name="Templates",Parent={10},}},
	{19,"Frame",{BackgroundColor3=Color3.new(0.19607844948769,0.19607844948769,0.19607844948769),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),Name="Players",Parent={18},Position=UDim2.new(0,0,0,25),Size=UDim2.new(1,0,0,86),Visible=false,ZIndex=10,}},
	{20,"TextLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Title",Parent={19},Size=UDim2.new(1,0,0,20),Text="Choose Players",TextColor3=Color3.new(1,1,1),TextSize=14,ZIndex=10,}},
	{21,"TextLabel",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="Any",Parent={19},Position=UDim2.new(0,5,0,42),Size=UDim2.new(1,-10,0,20),Text="Any Player",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{22,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="Button",Parent={21},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{23,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={22},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{24,"TextLabel",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="Me",Parent={19},Position=UDim2.new(0,5,0,20),Size=UDim2.new(1,-10,0,20),Text="Me Only",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{25,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="Button",Parent={24},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{26,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={25},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{27,"TextBox",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,ClearTextOnFocus=false,Font=3,Name="Custom",Parent={19},PlaceholderColor3=Color3.new(0.47058826684952,0.47058826684952,0.47058826684952),PlaceholderText="Custom Player Set",Position=UDim2.new(0,5,0,64),Size=UDim2.new(1,-35,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{28,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="CustomButton",Parent={19},Position=UDim2.new(1,-25,0,64),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{29,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={28},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{30,"Frame",{BackgroundColor3=Color3.new(0.19607844948769,0.19607844948769,0.19607844948769),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),Name="Strings",Parent={18},Position=UDim2.new(0,0,0,25),Size=UDim2.new(1,0,0,64),Visible=false,ZIndex=10,}},
	{31,"TextLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Title",Parent={30},Size=UDim2.new(1,0,0,20),Text="Choose String",TextColor3=Color3.new(1,1,1),TextSize=14,ZIndex=10,}},
	{32,"TextLabel",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="Any",Parent={30},Position=UDim2.new(0,5,0,20),Size=UDim2.new(1,-10,0,20),Text="Any String",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{33,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="Button",Parent={32},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{34,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={33},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{54,"Frame",{BackgroundColor3=Color3.new(0.19607844948769,0.19607844948769,0.19607844948769),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),Name="Numbers",Parent={18},Position=UDim2.new(0,0,0,25),Size=UDim2.new(1,0,0,64),Visible=false,ZIndex=10,}},
	{55,"TextLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Title",Parent={54},Size=UDim2.new(1,0,0,20),Text="Choose String",TextColor3=Color3.new(1,1,1),TextSize=14,ZIndex=10,}},
	{56,"TextLabel",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="Any",Parent={54},Position=UDim2.new(0,5,0,20),Size=UDim2.new(1,-10,0,20),Text="Any Number",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{57,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="Button",Parent={56},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{58,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={57},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{59,"TextBox",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,ClearTextOnFocus=false,Font=3,Name="Custom",Parent={54},PlaceholderColor3=Color3.new(0.47058826684952,0.47058826684952,0.47058826684952),PlaceholderText="Number",Position=UDim2.new(0,5,0,42),Size=UDim2.new(1,-35,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{60,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="CustomButton",Parent={54},Position=UDim2.new(1,-25,0,42),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{61,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={60},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{35,"TextBox",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,ClearTextOnFocus=false,Font=3,Name="Custom",Parent={30},PlaceholderColor3=Color3.new(0.47058826684952,0.47058826684952,0.47058826684952),PlaceholderText="Match String",Position=UDim2.new(0,5,0,42),Size=UDim2.new(1,-35,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{36,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="CustomButton",Parent={30},Position=UDim2.new(1,-25,0,42),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{37,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={36},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{38,"Frame",{BackgroundColor3=Color3.new(0.19607844948769,0.19607844948769,0.19607844948769),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),Name="DelayEditor",Parent={18},Position=UDim2.new(0,0,0,25),Size=UDim2.new(1,0,0,24),Visible=false,ZIndex=10,}},
	{39,"TextBox",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,Font=3,Name="Secs",Parent={38},PlaceholderColor3=Color3.new(0.47058826684952,0.47058826684952,0.47058826684952),Position=UDim2.new(0,60,0,2),Size=UDim2.new(1,-65,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{40,"TextLabel",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="Label",Parent={39},Position=UDim2.new(0,-55,0,0),Size=UDim2.new(1,0,1,0),Text="Delay (s):",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{41,"Frame",{BackgroundColor3=currentShade1,BorderSizePixel=0,ClipsDescendants=true,Name="EventTemplate",Parent={6},Size=UDim2.new(1,0,0,20),Visible=false,ZIndex=10,}},
	{42,"TextButton",{BackgroundColor3=currentText1,BackgroundTransparency=1,Font=3,Name="Expand",Parent={41},Size=UDim2.new(0,20,0,20),Text=">",TextColor3=Color3.new(1,1,1),TextSize=18,ZIndex=10,}},
	{43,"TextLabel",{BackgroundColor3=currentText1,BackgroundTransparency=1,Font=3,Name="EventName",Parent={41},Position=UDim2.new(0,25,0,0),Size=UDim2.new(1,-25,0,20),Text="OnSpawn",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{44,"Frame",{BackgroundColor3=Color3.new(0.19607844948769,0.19607844948769,0.19607844948769),BorderSizePixel=0,BackgroundTransparency=1,ClipsDescendants=true,Name="Cmds",Parent={41},Position=UDim2.new(0,0,0,20),Size=UDim2.new(1,0,1,-20),ZIndex=10,}},
	{45,"Frame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BorderColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),Name="Add",Parent={44},Position=UDim2.new(0,0,1,-20),Size=UDim2.new(1,0,0,20),ZIndex=10,}},
	{46,"TextBox",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,ClearTextOnFocus=false,Font=3,Parent={45},PlaceholderColor3=Color3.new(0.7843137383461,0.7843137383461,0.7843137383461),PlaceholderText="Add new command",Position=UDim2.new(0,5,0,0),Size=UDim2.new(1,-10,1,0),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{47,"Frame",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Name="Holder",Parent={44},Size=UDim2.new(1,0,1,-20),ZIndex=10,}},
	{48,"UIListLayout",{Parent={47},SortOrder=2,}},
	{49,"Frame",{currentShade1,BorderSizePixel=0,ClipsDescendants=true,Name="CmdTemplate",Parent={6},Size=UDim2.new(1,0,0,20),Visible=false,ZIndex=10,}},
	{50,"TextBox",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,ClearTextOnFocus=false,Font=3,Parent={49},PlaceholderColor3=Color3.new(1,1,1),Position=UDim2.new(0,5,0,0),Size=UDim2.new(1,-45,0,20),Text="a\\b\\c\\d",TextColor3=currentText1,TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{51,"TextButton",{BackgroundColor3=currentShade1,BorderSizePixel=0,Font=3,Name="Delete",Parent={49},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),Text="X",TextColor3=Color3.new(1,1,1),TextSize=18,ZIndex=10,}},
	{52,"TextButton",{BackgroundColor3=currentShade1,BorderSizePixel=0,Font=3,Name="Settings",Parent={49},Position=UDim2.new(1,-40,0,0),Size=UDim2.new(0,20,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=18,ZIndex=10,}},
	{53,"ImageLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Image="rbxassetid://1204397029",Parent={52},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),ZIndex=10,}},
})
main.Name = randomString()
local mainFrame = main:WaitForChild("Content")
local eventList = mainFrame:WaitForChild("List")
local eventListHolder = eventList:WaitForChild("Holder")
local cmdTemplate = mainFrame:WaitForChild("CmdTemplate")
local eventTemplate = mainFrame:WaitForChild("EventTemplate")
local settingsFrame = mainFrame:WaitForChild("Settings"):WaitForChild("Slider")
local settingsTemplates = mainFrame.Settings:WaitForChild("Templates")
local settingsList = settingsFrame:WaitForChild("List"):WaitForChild("Holder")
table.insert(shade2,main.TopBar) table.insert(shade1,mainFrame) table.insert(shade2,eventTemplate)
table.insert(text1,eventTemplate.EventName) table.insert(shade1,eventTemplate.Cmds.Add) table.insert(shade1,cmdTemplate)
table.insert(text1,cmdTemplate.TextBox) table.insert(shade2,cmdTemplate.Delete) table.insert(shade2,cmdTemplate.Settings)
table.insert(scroll,mainFrame.List) table.insert(shade1,settingsFrame) table.insert(shade2,settingsFrame.Line)
table.insert(shade2,settingsFrame.Close) table.insert(scroll,settingsFrame.List) table.insert(shade2,settingsTemplates.DelayEditor.Secs)
table.insert(text1,settingsTemplates.DelayEditor.Secs) table.insert(text1,settingsTemplates.DelayEditor.Secs.Label) table.insert(text1,settingsTemplates.Players.Title)
table.insert(shade3,settingsTemplates.Players.CustomButton) table.insert(shade2,settingsTemplates.Players.Custom) table.insert(text1,settingsTemplates.Players.Custom)
table.insert(shade3,settingsTemplates.Players.Any.Button) table.insert(shade3,settingsTemplates.Players.Me.Button) table.insert(text1,settingsTemplates.Players.Any)
table.insert(text1,settingsTemplates.Players.Me) table.insert(text1,settingsTemplates.Strings.Title) table.insert(text1,settingsTemplates.Strings.Any)
table.insert(shade3,settingsTemplates.Strings.Any.Button) table.insert(shade3,settingsTemplates.Strings.CustomButton) table.insert(text1,settingsTemplates.Strings.Custom)
table.insert(shade2,settingsTemplates.Strings.Custom)
table.insert(text1,settingsTemplates.Players.Me) table.insert(text1,settingsTemplates.Numbers.Title) table.insert(text1,settingsTemplates.Numbers.Any)
table.insert(shade3,settingsTemplates.Numbers.Any.Button) table.insert(shade3,settingsTemplates.Numbers.CustomButton) table.insert(text1,settingsTemplates.Numbers.Custom)
table.insert(shade2,settingsTemplates.Numbers.Custom)

local tweenInf = TweenInfo.new(0.25,Enum.EasingStyle.Quart,Enum.EasingDirection.Out)

local currentlyEditingCmd = nil

settingsFrame:WaitForChild("Close").MouseButton1Click:Connect(function()
	settingsFrame:TweenPosition(UDim2.new(0,-150,0,0),Enum.EasingDirection.Out,Enum.EasingStyle.Quart,0.25,true)
end)

local function resizeList()
	local size = 0

	for i,v in pairs(eventListHolder:GetChildren()) do
		if v.Name == "EventTemplate" then
			size = size + 20
			if v.Expand.Rotation == 90 then
				size = size + 20*(1+(#events[v.EventName:GetAttribute("RawName")].commands or 0))
			end
		end
	end

	TweenService:Create(eventList,tweenInf,{CanvasSize = UDim2.new(0,0,0,size)}):Play()

	if size > eventList.AbsoluteSize.Y then
		eventListHolder.Size = UDim2.new(1,-8,1,0)
	else
		eventListHolder.Size = UDim2.new(1,0,1,0)
	end
end

local function resizeSettingsList()
	local size = 0

	for i,v in pairs(settingsList:GetChildren()) do
		if v:IsA("Frame") then
			size = size + v.AbsoluteSize.Y
		end
	end

	settingsList.Parent.CanvasSize = UDim2.new(0,0,0,size)

	if size > settingsList.Parent.AbsoluteSize.Y then
		settingsList.Size = UDim2.new(1,-8,1,0)
	else
		settingsList.Size = UDim2.new(1,0,1,0)
	end
end

local function setupCheckbox(button,callback)
	local enabled = button.On.BackgroundTransparency == 0

	local function update()
		button.On.BackgroundTransparency = (enabled and 0 or 1)
	end

	button.On.MouseButton1Click:Connect(function()
		enabled = not enabled
		update()
		if callback then callback(enabled) end
	end)

	return {
		Toggle = function(nocall) enabled = not enabled update() if not nocall and callback then callback(enabled) end end,
		Enable = function(nocall) if enabled then return end enabled = true update()if not nocall and callback then callback(enabled) end end,
		Disable = function(nocall) if not enabled then return end enabled = false update()if not nocall and callback then callback(enabled) end end,
		IsEnabled = function() return enabled end
	}
end

local function openSettingsEditor(event,cmd)
	currentlyEditingCmd = cmd

	for i,v in pairs(settingsList:GetChildren()) do if v:IsA("Frame") then v:Destroy() end end

	local delayEditor = settingsTemplates.DelayEditor:Clone()
	delayEditor.Secs.FocusLost:Connect(function()
		cmd[3] = tonumber(delayEditor.Secs.Text) or 0
		delayEditor.Secs.Text = cmd[3]
		if onEdited then onEdited() end
	end)
	delayEditor.Secs.Text = cmd[3]
	delayEditor.Visible = true
	table.insert(shade2,delayEditor.Secs)
	table.insert(text1,delayEditor.Secs)
	table.insert(text1,delayEditor.Secs.Label)
	delayEditor.Parent = settingsList

	for i,v in pairs(event.sets) do
		if v.Type == "Player" then
			local template = settingsTemplates.Players:Clone()
			template.Title.Text = v.Name or "Player"

			local me,any,custom

			me = setupCheckbox(template.Me.Button,function(on)
				if not on then return end
				any.Disable()
				custom.Disable()
				cmd[2][i] = 0
				if onEdited then onEdited() end
			end)

			any = setupCheckbox(template.Any.Button,function(on)
				if not on then return end
				me.Disable()
				custom.Disable()
				cmd[2][i] = 1
				if onEdited then onEdited() end
			end)

			local customTextBox = template.Custom
			custom = setupCheckbox(template.CustomButton,function(on)
				if not on then return end
				me.Disable()
				any.Disable()
				cmd[2][i] = customTextBox.Text
				if onEdited then onEdited() end
			end)

			ViewportTextBox.convert(customTextBox)
			customTextBox.FocusLost:Connect(function()
				if custom:IsEnabled() then
					cmd[2][i] = customTextBox.Text
					if onEdited then onEdited() end
				end
			end)

			local cVal = cmd[2][i]
			if cVal == 0 then
				me:Enable()
			elseif cVal == 1 then
				any:Enable()
			else
				custom:Enable()
				customTextBox.Text = cVal
			end

			template.Visible = true
			table.insert(text1,template.Title)
			table.insert(shade3,template.CustomButton)
			table.insert(shade3,template.Any.Button)
			table.insert(shade3,template.Me.Button)
			table.insert(text1,template.Any)
			table.insert(text1,template.Me)
			template.Parent = settingsList
		elseif v.Type == "String" then
			local template = settingsTemplates.Strings:Clone()
			template.Title.Text = v.Name or "String"

			local any,custom

			any = setupCheckbox(template.Any.Button,function(on)
				if not on then return end
				custom.Disable()
				cmd[2][i] = 0
				if onEdited then onEdited() end
			end)

			local customTextBox = template.Custom
			custom = setupCheckbox(template.CustomButton,function(on)
				if not on then return end
				any.Disable()
				cmd[2][i] = customTextBox.Text
				if onEdited then onEdited() end
			end)

			ViewportTextBox.convert(customTextBox)
			customTextBox.FocusLost:Connect(function()
				if custom:IsEnabled() then
					cmd[2][i] = customTextBox.Text
					if onEdited then onEdited() end
				end
			end)

			local cVal = cmd[2][i]
			if cVal == 0 then
				any:Enable()
			else
				custom:Enable()
				customTextBox.Text = cVal
			end

			template.Visible = true
			table.insert(text1,template.Title)
			table.insert(text1,template.Any)
			table.insert(shade3,template.Any.Button)
			table.insert(shade3,template.CustomButton)
			template.Parent = settingsList
		elseif v.Type == "Number" then
			local template = settingsTemplates.Numbers:Clone()
			template.Title.Text = v.Name or "Number"

			local any,custom

			any = setupCheckbox(template.Any.Button,function(on)
				if not on then return end
				custom.Disable()
				cmd[2][i] = 0
				if onEdited then onEdited() end
			end)

			local customTextBox = template.Custom
			custom = setupCheckbox(template.CustomButton,function(on)
				if not on then return end
				any.Disable()
				cmd[2][i] = customTextBox.Text
				if onEdited then onEdited() end
			end)

			ViewportTextBox.convert(customTextBox)
			customTextBox.FocusLost:Connect(function()
				cmd[2][i] = tonumber(customTextBox.Text) or 0
				customTextBox.Text = cmd[2][i]
				if custom:IsEnabled() then
					if onEdited then onEdited() end
				end
			end)

			local cVal = cmd[2][i]
			if cVal == 0 then
				any:Enable()
			else
				custom:Enable()
				customTextBox.Text = cVal
			end

			template.Visible = true
			table.insert(text1,template.Title)
			table.insert(text1,template.Any)
			table.insert(shade3,template.Any.Button)
			table.insert(shade3,template.CustomButton)
			template.Parent = settingsList
		end
	end
	resizeSettingsList()
	settingsFrame:TweenPosition(UDim2.new(0,0,0,0),Enum.EasingDirection.Out,Enum.EasingStyle.Quart,0.25,true)
end

local function defaultSettings(ev)
	local res = {}

	for i,v in pairs(ev.sets) do
		if v.Type == "Player" then
			res[#res+1] = v.Default or 0
		elseif v.Type == "String" then
			res[#res+1] = v.Default or 0
		elseif v.Type == "Number" then
			res[#res+1] = v.Default or 0
		end
	end

	return res
end

local function refreshList()
	for i,v in pairs(eventListHolder:GetChildren()) do if v:IsA("Frame") then v:Destroy() end end

	for name,event in pairs(events) do
		local eventF = eventTemplate:Clone()
		eventF.EventName.Text = name
		eventF.Visible = true
		eventF.EventName:SetAttribute("RawName", name)
		table.insert(shade2,eventF)
		table.insert(text1,eventF.EventName)
		table.insert(shade1,eventF.Cmds.Add)

		local expanded = false
		eventF.Expand.MouseButton1Down:Connect(function()
			expanded = not expanded
			eventF:TweenSize(UDim2.new(1,0,0,20 + (expanded and 20*#eventF.Cmds.Holder:GetChildren() or 0)),Enum.EasingDirection.Out,Enum.EasingStyle.Quart,0.25,true)
			eventF.Expand.Rotation = expanded and 90 or 0
			resizeList()
		end)

		local function refreshCommands()
			for i,v in pairs(eventF.Cmds.Holder:GetChildren()) do
				if v.Name == "CmdTemplate" then
					v:Destroy()
				end
			end

			eventF.EventName.Text = name..(#event.commands > 0 and " ("..#event.commands..")" or "")

			for i,cmd in pairs(event.commands) do
				local cmdF = cmdTemplate:Clone()
				local cmdTextBox = cmdF.TextBox
				ViewportTextBox.convert(cmdTextBox)
				cmdTextBox.Text = cmd[1]
				cmdF.Visible = true
				table.insert(shade1,cmdF)
				table.insert(shade2,cmdF.Delete)
				table.insert(shade2,cmdF.Settings)

				cmdTextBox.FocusLost:Connect(function()
					event.commands[i] = {cmdTextBox.Text,cmd[2],cmd[3]}
					if onEdited then onEdited() end
				end)

				cmdF.Settings.MouseButton1Click:Connect(function()
					openSettingsEditor(event,cmd)
				end)

				cmdF.Delete.MouseButton1Click:Connect(function()
					table.remove(event.commands,i)
					refreshCommands()
					resizeList()

					if currentlyEditingCmd == cmd then
						settingsFrame:TweenPosition(UDim2.new(0,-150,0,0),Enum.EasingDirection.Out,Enum.EasingStyle.Quart,0.25,true)
					end
					if onEdited then onEdited() end
				end)

				cmdF.Parent = eventF.Cmds.Holder
			end

			eventF:TweenSize(UDim2.new(1,0,0,20 + (expanded and 20*#eventF.Cmds.Holder:GetChildren() or 0)),Enum.EasingDirection.Out,Enum.EasingStyle.Quart,0.25,true)
		end

		local newBox = eventF.Cmds.Add.TextBox
		ViewportTextBox.convert(newBox)
		newBox.FocusLost:Connect(function(enter)
			if enter then
				event.commands[#event.commands+1] = {newBox.Text,defaultSettings(event),0}
				newBox.Text = ""

				refreshCommands()
				resizeList()
				if onEdited then onEdited() end
			end
		end)

		--eventF:GetPropertyChangedSignal("AbsoluteSize"):Connect(resizeList)

		eventF.Parent = eventListHolder

		refreshCommands()
	end

	resizeList()
end

local function saveData()
	local result = {}
	for i,v in pairs(events) do
		result[i] = v.commands
	end
	return HttpService:JSONEncode(result)
end

local function loadData(str)
	local data = HttpService:JSONDecode(str)
	for i,v in pairs(data) do
		if events[i] then
			events[i].commands = v
		end
	end
end

local function addCmd(event,data)
	table.insert(events[event].commands,data)
end

local function setOnEdited(f)
	if type(f) == "function" then
		onEdited = f
	end
end

main.TopBar.Close.MouseButton1Click:Connect(function()
	main:TweenPosition(UDim2.new(0.5,-175,0,-500), "InOut", "Quart", 0.5, true, nil)
end)
dragGUI(main)
main.Parent = PARENT

return {
	RegisterEvent = registerEvent,
	FireEvent = fireEvent,
	Refresh = refreshList,
	SaveData = saveData,
	LoadData = loadData,
	AddCmd =ddCmd,
	Frame = ma

local function registerEvent(name,sets)
	events[name] = {
		commands = {},
		sets = sets or {}
	}
end

local onEdited = nil

local function fireEvent(name,...)
	local args = {...}
	local event = events[name]
	if event then
		for i,cmd in pairs(event.commands) do
			local metCondition = true
			for idx,set in pairs(event.sets) do
				local argVal = args[idx]
				local cmdSet = cmd[2][idx]
				local condType = set.Type
				if condType == "Player" then
					if cmdSet == 0 then
						metCondition = metCondition and (tostring(Players.LocalPlayer) == argVal)
					elseif cmdSet ~= 1 then
						metCondition = metCondition and table.find(getPlayer(cmdSet,Players.LocalPlayer),argVal)
					end
				elseif condType == "String" then
					if cmdSet ~= 0 then
						metCondition = metCondition and string.find(argVal:lower(),cmdSet:lower())
					end
				elseif condType == "Number" then
					if cmdSet ~= 0 then
						metCondition = metCondition and tonumber(argVal)<=tonumber(cmdSet)
					end
				end
				if not metCondition then break end
			end

			if metCondition then
				pcall(task.spawn(function()
					local cmdStr = cmd[1]
					for count,arg in pairs(args) do
						cmdStr = cmdStr:gsub("%$"..count,arg)
					end
					wait(cmd[3] or 0)
					execCmd(cmdStr)
				end))
			end
		end
	end
end

local main = create({
	{1,"Frame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BackgroundTransparency=1,BorderSizePixel=0,Name="EventEditor",Position=UDim2.new(0.5,-175,0,-500),Size=UDim2.new(0,350,0,20),ZIndex=10,}},
	{2,"Frame",{BackgroundColor3=currentShade2,BorderSizePixel=0,Name="TopBar",Parent={1},Size=UDim2.new(1,0,0,20),ZIndex=10,}},
	{3,"TextLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Title",Parent={2},Position=UDim2.new(0,0,0,0),Size=UDim2.new(1,0,0.95,0),Text="Event Editor",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=Enum.TextXAlignment.Center,ZIndex=10,}},
	{4,"TextButton",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Close",Parent={2},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,ZIndex=10,}},
	{5,"ImageLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Image="rbxassetid://5054663650",Parent={4},Position=UDim2.new(0,5,0,5),Size=UDim2.new(0,10,0,10),ZIndex=10,}},
	{6,"Frame",{BackgroundColor3=currentShade1,BorderSizePixel=0,Name="Content",Parent={1},Position=UDim2.new(0,0,0,20),Size=UDim2.new(1,0,0,202),ZIndex=10,}},
	{7,"ScrollingFrame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,BottomImage="rbxasset://textures/ui/Scroll/scroll-middle.png",CanvasSize=UDim2.new(0,0,0,100),Name="List",Parent={6},Position=UDim2.new(0,5,0,5),ScrollBarImageColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),ScrollBarThickness=8,Size=UDim2.new(1,-10,1,-10),TopImage="rbxasset://textures/ui/Scroll/scroll-middle.png",ZIndex=10,}},
	{8,"Frame",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Name="Holder",Parent={7},Size=UDim2.new(1,0,1,0),ZIndex=10,}},
	{9,"UIListLayout",{Parent={8},SortOrder=2,}},
	{10,"Frame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BackgroundTransparency=1,BorderColor3=Color3.new(0.3137255012989,0.3137255012989,0.3137255012989),BorderSizePixel=0,ClipsDescendants=true,Name="Settings",Parent={6},Position=UDim2.new(1,0,0,0),Size=UDim2.new(0,150,1,0),ZIndex=10,}},
	{11,"Frame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),Name="Slider",Parent={10},Position=UDim2.new(0,-150,0,0),Size=UDim2.new(1,0,1,0),ZIndex=10,}},
	{12,"Frame",{BackgroundColor3=Color3.new(0.23529413342476,0.23529413342476,0.23529413342476),BorderColor3=Color3.new(0.3137255012989,0.3137255012989,0.3137255012989),BorderSizePixel=0,Name="Line",Parent={11},Size=UDim2.new(0,1,1,0),ZIndex=10,}},
	{13,"ScrollingFrame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,BottomImage="rbxasset://textures/ui/Scroll/scroll-middle.png",CanvasSize=UDim2.new(0,0,0,100),Name="List",Parent={11},Position=UDim2.new(0,0,0,25),ScrollBarImageColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),ScrollBarThickness=8,Size=UDim2.new(1,0,1,-25),TopImage="rbxasset://textures/ui/Scroll/scroll-middle.png",ZIndex=10,}},
	{14,"Frame",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Name="Holder",Parent={13},Size=UDim2.new(1,0,1,0),ZIndex=10,}},
	{15,"UIListLayout",{Parent={14},SortOrder=2,}},
	{16,"TextLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Title",Parent={11},Size=UDim2.new(1,0,0,20),Text="Event Settings",TextColor3=Color3.new(1,1,1),TextSize=14,ZIndex=10,}},
	{17,"TextButton",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),Font=3,Name="Close",BorderSizePixel=0,Parent={11},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),Text="<",TextColor3=Color3.new(1,1,1),TextSize=18,ZIndex=10,}},
	{18,"Folder",{Name="Templates",Parent={10},}},
	{19,"Frame",{BackgroundColor3=Color3.new(0.19607844948769,0.19607844948769,0.19607844948769),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),Name="Players",Parent={18},Position=UDim2.new(0,0,0,25),Size=UDim2.new(1,0,0,86),Visible=false,ZIndex=10,}},
	{20,"TextLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Title",Parent={19},Size=UDim2.new(1,0,0,20),Text="Choose Players",TextColor3=Color3.new(1,1,1),TextSize=14,ZIndex=10,}},
	{21,"TextLabel",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="Any",Parent={19},Position=UDim2.new(0,5,0,42),Size=UDim2.new(1,-10,0,20),Text="Any Player",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{22,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="Button",Parent={21},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{23,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={22},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{24,"TextLabel",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="Me",Parent={19},Position=UDim2.new(0,5,0,20),Size=UDim2.new(1,-10,0,20),Text="Me Only",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{25,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="Button",Parent={24},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{26,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={25},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{27,"TextBox",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,ClearTextOnFocus=false,Font=3,Name="Custom",Parent={19},PlaceholderColor3=Color3.new(0.47058826684952,0.47058826684952,0.47058826684952),PlaceholderText="Custom Player Set",Position=UDim2.new(0,5,0,64),Size=UDim2.new(1,-35,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{28,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="CustomButton",Parent={19},Position=UDim2.new(1,-25,0,64),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{29,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={28},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{30,"Frame",{BackgroundColor3=Color3.new(0.19607844948769,0.19607844948769,0.19607844948769),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),Name="Strings",Parent={18},Position=UDim2.new(0,0,0,25),Size=UDim2.new(1,0,0,64),Visible=false,ZIndex=10,}},
	{31,"TextLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Title",Parent={30},Size=UDim2.new(1,0,0,20),Text="Choose String",TextColor3=Color3.new(1,1,1),TextSize=14,ZIndex=10,}},
	{32,"TextLabel",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="Any",Parent={30},Position=UDim2.new(0,5,0,20),Size=UDim2.new(1,-10,0,20),Text="Any String",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{33,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="Button",Parent={32},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{34,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={33},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{54,"Frame",{BackgroundColor3=Color3.new(0.19607844948769,0.19607844948769,0.19607844948769),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),Name="Numbers",Parent={18},Position=UDim2.new(0,0,0,25),Size=UDim2.new(1,0,0,64),Visible=false,ZIndex=10,}},
	{55,"TextLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Title",Parent={54},Size=UDim2.new(1,0,0,20),Text="Choose String",TextColor3=Color3.new(1,1,1),TextSize=14,ZIndex=10,}},
	{56,"TextLabel",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="Any",Parent={54},Position=UDim2.new(0,5,0,20),Size=UDim2.new(1,-10,0,20),Text="Any Number",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{57,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="Button",Parent={56},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{58,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={57},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{59,"TextBox",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,ClearTextOnFocus=false,Font=3,Name="Custom",Parent={54},PlaceholderColor3=Color3.new(0.47058826684952,0.47058826684952,0.47058826684952),PlaceholderText="Number",Position=UDim2.new(0,5,0,42),Size=UDim2.new(1,-35,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{60,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="CustomButton",Parent={54},Position=UDim2.new(1,-25,0,42),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{61,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={60},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{35,"TextBox",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,ClearTextOnFocus=false,Font=3,Name="Custom",Parent={30},PlaceholderColor3=Color3.new(0.47058826684952,0.47058826684952,0.47058826684952),PlaceholderText="Match String",Position=UDim2.new(0,5,0,42),Size=UDim2.new(1,-35,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{36,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="CustomButton",Parent={30},Position=UDim2.new(1,-25,0,42),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{37,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={36},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{38,"Frame",{BackgroundColor3=Color3.new(0.19607844948769,0.19607844948769,0.19607844948769),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),Name="DelayEditor",Parent={18},Position=UDim2.new(0,0,0,25),Size=UDim2.new(1,0,0,24),Visible=false,ZIndex=10,}},
	{39,"TextBox",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,Font=3,Name="Secs",Parent={38},PlaceholderColor3=Color3.new(0.47058826684952,0.47058826684952,0.47058826684952),Position=UDim2.new(0,60,0,2),Size=UDim2.new(1,-65,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{40,"TextLabel",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="Label",Parent={39},Position=UDim2.new(0,-55,0,0),Size=UDim2.new(1,0,1,0),Text="Delay (s):",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{41,"Frame",{BackgroundColor3=currentShade1,BorderSizePixel=0,ClipsDescendants=true,Name="EventTemplate",Parent={6},Size=UDim2.new(1,0,0,20),Visible=false,ZIndex=10,}},
	{42,"TextButton",{BackgroundColor3=currentText1,BackgroundTransparency=1,Font=3,Name="Expand",Parent={41},Size=UDim2.new(0,20,0,20),Text=">",TextColor3=Color3.new(1,1,1),TextSize=18,ZIndex=10,}},
	{43,"TextLabel",{BackgroundColor3=currentText1,BackgroundTransparency=1,Font=3,Name="EventName",Parent={41},Position=UDim2.new(0,25,0,0),Size=UDim2.new(1,-25,0,20),Text="OnSpawn",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{44,"Frame",{BackgroundColor3=Color3.new(0.19607844948769,0.19607844948769,0.19607844948769),BorderSizePixel=0,BackgroundTransparency=1,ClipsDescendants=true,Name="Cmds",Parent={41},Position=UDim2.new(0,0,0,20),Size=UDim2.new(1,0,1,-20),ZIndex=10,}},
	{45,"Frame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BorderColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),Name="Add",Parent={44},Position=UDim2.new(0,0,1,-20),Size=UDim2.new(1,0,0,20),ZIndex=10,}},
	{46,"TextBox",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,ClearTextOnFocus=false,Font=3,Parent={45},PlaceholderColor3=Color3.new(0.7843137383461,0.7843137383461,0.7843137383461),PlaceholderText="Add new command",Position=UDim2.new(0,5,0,0),Size=UDim2.new(1,-10,1,0),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{47,"Frame",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Name="Holder",Parent={44},Size=UDim2.new(1,0,1,-20),ZIndex=10,}},
	{48,"UIListLayout",{Parent={47},SortOrder=2,}},
	{49,"Frame",{currentShade1,BorderSizePixel=0,ClipsDescendants=true,Name="CmdTemplate",Parent={6},Size=UDim2.new(1,0,0,20),Visible=false,ZIndex=10,}},
	{50,"TextBox",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,ClearTextOnFocus=false,Font=3,Parent={49},PlaceholderColor3=Color3.new(1,1,1),Position=UDim2.new(0,5,0,0),Size=UDim2.new(1,-45,0,20),Text="a\\b\\c\\d",TextColor3=currentText1,TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{51,"TextButton",{BackgroundColor3=currentShade1,BorderSizePixel=0,Font=3,Name="Delete",Parent={49},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),Text="X",TextColor3=Color3.new(1,1,1),TextSize=18,ZIndex=10,}},
	{52,"TextButton",{BackgroundColor3=currentShade1,BorderSizePixel=0,Font=3,Name="Settings",Parent={49},Position=UDim2.new(1,-40,0,0),Size=UDim2.new(0,20,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=18,ZIndex=10,}},
	{53,"ImageLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Image="rbxassetid://1204397029",Parent={52},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),ZIndex=10,}},
})
main.Name = randomString()
local mainFrame = main:WaitForChild("Content")
local eventList = mainFrame:WaitForChild("List")
local eventListHolder = eventList:WaitForChild("Holder")
local cmdTemplate = mainFrame:WaitForChild("CmdTemplate")
local eventTemplate = mainFrame:WaitForChild("EventTemplate")
local settingsFrame = mainFrame:WaitForChild("Settings"):WaitForChild("Slider")
local settingsTemplates = mainFrame.Settings:WaitForChild("Templates")
local settingsList = settingsFrame:WaitForChild("List"):WaitForChild("Holder")
table.insert(shade2,main.TopBar) table.insert(shade1,mainFrame) table.insert(shade2,eventTemplate)
table.insert(text1,eventTemplate.EventName) table.insert(shade1,eventTemplate.Cmds.Add) table.insert(shade1,cmdTemplate)
table.insert(text1,cmdTemplate.TextBox) table.insert(shade2,cmdTemplate.Delete) table.insert(shade2,cmdTemplate.Settings)
table.insert(scroll,mainFrame.List) table.insert(shade1,settingsFrame) table.insert(shade2,settingsFrame.Line)
table.insert(shade2,settingsFrame.Close) table.insert(scroll,settingsFrame.List) table.insert(shade2,settingsTemplates.DelayEditor.Secs)
table.insert(text1,settingsTemplates.DelayEditor.Secs) table.insert(text1,settingsTemplates.DelayEditor.Secs.Label) table.insert(text1,settingsTemplates.Players.Title)
table.insert(shade3,settingsTemplates.Players.CustomButton) table.insert(shade2,settingsTemplates.Players.Custom) table.insert(text1,settingsTemplates.Players.Custom)
table.insert(shade3,settingsTemplates.Players.Any.Button) table.insert(shade3,settingsTemplates.Players.Me.Button) table.insert(text1,settingsTemplates.Players.Any)
table.insert(text1,settingsTemplates.Players.Me) table.insert(text1,settingsTemplates.Strings.Title) table.insert(text1,settingsTemplates.Strings.Any)
table.insert(shade3,settingsTemplates.Strings.Any.Button) table.insert(shade3,settingsTemplates.Strings.CustomButton) table.insert(text1,settingsTemplates.Strings.Custom)
table.insert(shade2,settingsTemplates.Strings.Custom)
table.insert(text1,settingsTemplates.Players.Me) table.insert(text1,settingsTemplates.Numbers.Title) table.insert(text1,settingsTemplates.Numbers.Any)
table.insert(shade3,settingsTemplates.Numbers.Any.Button) table.insert(shade3,settingsTemplates.Numbers.CustomButton) table.insert(text1,settingsTemplates.Numbers.Custom)
table.insert(shade2,settingsTemplates.Numbers.Custom)

local tweenInf = TweenInfo.new(0.25,Enum.EasingStyle.Quart,Enum.EasingDirection.Out)

local currentlyEditingCmd = nil

settingsFrame:WaitForChild("Close").MouseButton1Click:Connect(function()
	settingsFrame:TweenPosition(UDim2.new(0,-150,0,0),Enum.EasingDirection.Out,Enum.EasingStyle.Quart,0.25,true)
end)

local function resizeList()
	local size = 0

	for i,v in pairs(eventListHolder:GetChildren()) do
		if v.Name == "EventTemplate" then
			size = size + 20
			if v.Expand.Rotation == 90 then
				size = size + 20*(1+(#events[v.EventName:GetAttribute("RawName")].commands or 0))
			end
		end
	end

	TweenService:Create(eventList,tweenInf,{CanvasSize = UDim2.new(0,0,0,size)}):Play()

	if size > eventList.AbsoluteSize.Y then
		eventListHolder.Size = UDim2.new(1,-8,1,0)
	else
		eventListHolder.Size = UDim2.new(1,0,1,0)
	end
end

local function resizeSettingsList()
	local size = 0

	for i,v in pairs(settingsList:GetChildren()) do
		if v:IsA("Frame") then
			size = size + v.AbsoluteSize.Y
		end
	end

	settingsList.Parent.CanvasSize = UDim2.new(0,0,0,size)

	if size > settingsList.Parent.AbsoluteSize.Y then
		settingsList.Size = UDim2.new(1,-8,1,0)
	else
		settingsList.Size = UDim2.new(1,0,1,0)
	end
end

local function setupCheckbox(button,callback)
	local enabled = button.On.BackgroundTransparency == 0

	local function update()
		button.On.BackgroundTransparency = (enabled and 0 or 1)
	end

	button.On.MouseButton1Click:Connect(function()
		enabled = not enabled
		update()
		if callback then callback(enabled) end
	end)

	return {
		Toggle = function(nocall) enabled = not enabled update() if not nocall and callback then callback(enabled) end end,
		Enable = function(nocall) if enabled then return end enabled = true update()if not nocall and callback then callback(enabled) end end,
		Disable = function(nocall) if not enabled then return end enabled = false update()if not nocall and callback then callback(enabled) end end,
		IsEnabled = function() return enabled end
	}
end

local function openSettingsEditor(event,cmd)
	currentlyEditingCmd = cmd

	for i,v in pairs(settingsList:GetChildren()) do if v:IsA("Frame") then v:Destroy() end end

	local delayEditor = settingsTemplates.DelayEditor:Clone()
	delayEditor.Secs.FocusLost:Connect(function()
		cmd[3] = tonumber(delayEditor.Secs.Text) or 0
		delayEditor.Secs.Text = cmd[3]
		if onEdited then onEdited() end
	end)
	delayEditor.Secs.Text = cmd[3]
	delayEditor.Visible = true
	table.insert(shade2,delayEditor.Secs)
	table.insert(text1,delayEditor.Secs)
	table.insert(text1,delayEditor.Secs.Label)
	delayEditor.Parent = settingsList

	for i,v in pairs(event.sets) do
		if v.Type == "Player" then
			local template = settingsTemplates.Players:Clone()
			template.Title.Text = v.Name or "Player"

			local me,any,custom

			me = setupCheckbox(template.Me.Button,function(on)
				if not on then return end
				any.Disable()
				custom.Disable()
				cmd[2][i] = 0
				if onEdited then onEdited() end
			end)

			any = setupCheckbox(template.Any.Button,function(on)
				if not on then return end
				me.Disable()
				custom.Disable()
				cmd[2][i] = 1
				if onEdited then onEdited() end
			end)

			local customTextBox = template.Custom
			custom = setupCheckbox(template.CustomButton,function(on)
				if not on then return end
				me.Disable()
				any.Disable()
				cmd[2][i] = customTextBox.Text
				if onEdited then onEdited() end
			end)

			ViewportTextBox.convert(customTextBox)
			customTextBox.FocusLost:Connect(function()
				if custom:IsEnabled() then
					cmd[2][i] = customTextBox.Text
					if onEdited then onEdited() end
				end
			end)

			local cVal = cmd[2][i]
			if cVal == 0 then
				me:Enable()
			elseif cVal == 1 then
				any:Enable()
			else
				custom:Enable()
				customTextBox.Text = cVal
			end

			template.Visible = true
			table.insert(text1,template.Title)
			table.insert(shade3,template.CustomButton)
			table.insert(shade3,template.Any.Button)
			table.insert(shade3,template.Me.Button)
			table.insert(text1,template.Any)
			table.insert(text1,template.Me)
			template.Parent = settingsList
		elseif v.Type == "String" then
			local template = settingsTemplates.Strings:Clone()
			template.Title.Text = v.Name or "String"

			local any,custom

			any = setupCheckbox(template.Any.Button,function(on)
				if not on then return end
				custom.Disable()
				cmd[2][i] = 0
				if onEdited then onEdited() end
			end)

			local customTextBox = template.Custom
			custom = setupCheckbox(template.CustomButton,function(on)
				if not on then return end
				any.Disable()
				cmd[2][i] = customTextBox.Text
				if onEdited then onEdited() end
			end)

			ViewportTextBox.convert(customTextBox)
			customTextBox.FocusLost:Connect(function()
				if custom:IsEnabled() then
					cmd[2][i] = customTextBox.Text
					if onEdited then onEdited() end
				end
			end)

			local cVal = cmd[2][i]
			if cVal == 0 then
				any:Enable()
			else
				custom:Enable()
				customTextBox.Text = cVal
			end

			template.Visible = true
			table.insert(text1,template.Title)
			table.insert(text1,template.Any)
			table.insert(shade3,template.Any.Button)
			table.insert(shade3,template.CustomButton)
			template.Parent = settingsList
		elseif v.Type == "Number" then
			local template = settingsTemplates.Numbers:Clone()
			template.Title.Text = v.Name or "Number"

			local any,custom

			any = setupCheckbox(template.Any.Button,function(on)
				if not on then return end
				custom.Disable()
				cmd[2][i] = 0
				if onEdited then onEdited() end
			end)

			local customTextBox = template.Custom
			custom = setupCheckbox(template.CustomButton,function(on)
				if not on then return end
				any.Disable()
				cmd[2][i] = customTextBox.Text
				if onEdited then onEdited() end
			end)

			ViewportTextBox.convert(customTextBox)
			customTextBox.FocusLost:Connect(function()
				cmd[2][i] = tonumber(customTextBox.Text) or 0
				customTextBox.Text = cmd[2][i]
				if custom:IsEnabled() then
					if onEdited then onEdited() end
				end
			end)

			local cVal = cmd[2][i]
			if cVal == 0 then
				any:Enable()
			else
				custom:Enable()
				customTextBox.Text = cVal
			end

			template.Visible = true
			table.insert(text1,template.Title)
			table.insert(text1,template.Any)
			table.insert(shade3,template.Any.Button)
			table.insert(shade3,template.CustomButton)
			template.Parent = settingsList
		end
	end
	resizeSettingsList()
	settingsFrame:TweenPosition(UDim2.new(0,0,0,0),Enum.EasingDirection.Out,Enum.EasingStyle.Quart,0.25,true)
end

local function defaultSettings(ev)
	local res = {}

	for i,v in pairs(ev.sets) do
		if v.Type == "Player" then
			res[#res+1] = v.Default or 0
		elseif v.Type == "String" then
			res[#res+1] = v.Default or 0
		elseif v.Type == "Number" then
			res[#res+1] = v.Default or 0
		end
	end

	return res
end

local function refreshList()
	for i,v in pairs(eventListHolder:GetChildren()) do if v:IsA("Frame") then v:Destroy() end end

	for name,event in pairs(events) do
		local eventF = eventTemplate:Clone()
		eventF.EventName.Text = name
		eventF.Visible = true
		eventF.EventName:SetAttribute("RawName", name)
		table.insert(shade2,eventF)
		table.insert(text1,eventF.EventName)
		table.insert(shade1,eventF.Cmds.Add)

		local expanded = false
		eventF.Expand.MouseButton1Down:Connect(function()
			expanded = not expanded
			eventF:TweenSize(UDim2.new(1,0,0,20 + (expanded and 20*#eventF.Cmds.Holder:GetChildren() or 0)),Enum.EasingDirection.Out,Enum.EasingStyle.Quart,0.25,true)
			eventF.Expand.Rotation = expanded and 90 or 0
			resizeList()
		end)

		local function refreshCommands()
			for i,v in pairs(eventF.Cmds.Holder:GetChildren()) do
				if v.Name == "CmdTemplate" then
					v:Destroy()
				end
			end

			eventF.EventName.Text = name..(#event.commands > 0 and " ("..#event.commands..")" or "")

			for i,cmd in pairs(event.commands) do
				local cmdF = cmdTemplate:Clone()
				local cmdTextBox = cmdF.TextBox
				ViewportTextBox.convert(cmdTextBox)
				cmdTextBox.Text = cmd[1]
				cmdF.Visible = true
				table.insert(shade1,cmdF)
				table.insert(shade2,cmdF.Delete)
				table.insert(shade2,cmdF.Settings)

				cmdTextBox.FocusLost:Connect(function()
					event.commands[i] = {cmdTextBox.Text,cmd[2],cmd[3]}
					if onEdited then onEdited() end
				end)

				cmdF.Settings.MouseButton1Click:Connect(function()
					openSettingsEditor(event,cmd)
				end)

				cmdF.Delete.MouseButton1Click:Connect(function()
					table.remove(event.commands,i)
					refreshCommands()
					resizeList()

					if currentlyEditingCmd == cmd then
						settingsFrame:TweenPosition(UDim2.new(0,-150,0,0),Enum.EasingDirection.Out,Enum.EasingStyle.Quart,0.25,true)
					end
					if onEdited then onEdited() end
				end)

				cmdF.Parent = eventF.Cmds.Holder
			end

			eventF:TweenSize(UDim2.new(1,0,0,20 + (expanded and 20*#eventF.Cmds.Holder:GetChildren() or 0)),Enum.EasingDirection.Out,Enum.EasingStyle.Quart,0.25,true)
		end

		local newBox = eventF.Cmds.Add.TextBox
		ViewportTextBox.convert(newBox)
		newBox.FocusLost:Connect(function(enter)
			if enter then
				event.commands[#event.commands+1] = {newBox.Text,defaultSettings(event),0}
				newBox.Text = ""

				refreshCommands()
				resizeList()
				if onEdited then onEdited() end
			end
		end)

		--eventF:GetPropertyChangedSignal("AbsoluteSize"):Connect(resizeList)

		eventF.Parent = eventListHolder

		refreshCommands()
	end

	resizeList()
end

local function saveData()
	local result = {}
	for i,v in pairs(events) do
		result[i] = v.commands
	end
	return HttpService:JSONEncode(result)
end

local function loadData(str)
	local data = HttpService:JSONDecode(str)
	for i,v in pairs(data) do
		if events[i] then
			events[i].commands = v
		end
	end
end

local function addCmd(event,data)
	table.insert(events[event].commands,data)
end

local function setOnEdited(f)
	if type(f) == "function" then
		onEdited = f
	end
end

main.TopBar.Close.MouseButton1Click:Connect(function()
	main:TweenPosition(UDim2.new(0.5,-175,0,-500), "InOut", "Quart", 0.5, true, nil)
end)
dragGUI(main)
main.Parent = PARENT

return {
	RegisterEvent = registerEvent,
	FireEvent = fireEvent,
	Refresh = refreshList,
	SaveData = saveData,
	LoadData = loadData,
	AddCmd = addCmd,
	Frame = main,
	SetOnEdited = setOnEdited
}

local function registerEvent(name,sets)
	events[name] = {
		commands = {},
		sets = sets or {}
	}
end

local onEdited = nil

local function fireEvent(name,...)
	local args = {...}
	local event = events[name]
	if event then
		for i,cmd in pairs(event.commands) do
			local metCondition = true
			for idx,set in pairs(event.sets) do
				local argVal = args[idx]
				local cmdSet = cmd[2][idx]
				local condType = set.Type
				if condType == "Player" then
					if cmdSet == 0 then
						metCondition = metCondition and (tostring(Players.LocalPlayer) == argVal)
					elseif cmdSet ~= 1 then
						metCondition = metCondition and table.find(getPlayer(cmdSet,Players.LocalPlayer),argVal)
					end
				elseif condType == "String" then
					if cmdSet ~= 0 then
						metCondition = metCondition and string.find(argVal:lower(),cmdSet:lower())
					end
				elseif condType == "Number" then
					if cmdSet ~= 0 then
						metCondition = metCondition and tonumber(argVal)<=tonumber(cmdSet)
					end
				end
				if not metCondition then break end
			end

			if metCondition then
				pcall(task.spawn(function()
					local cmdStr = cmd[1]
					for count,arg in pairs(args) do
						cmdStr = cmdStr:gsub("%$"..count,arg)
					end
					wait(cmd[3] or 0)
					execCmd(cmdStr)
				end))
			end
		end
	end
end

local main = create({
	{1,"Frame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BackgroundTransparency=1,BorderSizePixel=0,Name="EventEditor",Position=UDim2.new(0.5,-175,0,-500),Size=UDim2.new(0,350,0,20),ZIndex=10,}},
	{2,"Frame",{BackgroundColor3=currentShade2,BorderSizePixel=0,Name="TopBar",Parent={1},Size=UDim2.new(1,0,0,20),ZIndex=10,}},
	{3,"TextLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Title",Parent={2},Position=UDim2.new(0,0,0,0),Size=UDim2.new(1,0,0.95,0),Text="Event Editor",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=Enum.TextXAlignment.Center,ZIndex=10,}},
	{4,"TextButton",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Close",Parent={2},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,ZIndex=10,}},
	{5,"ImageLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Image="rbxassetid://5054663650",Parent={4},Position=UDim2.new(0,5,0,5),Size=UDim2.new(0,10,0,10),ZIndex=10,}},
	{6,"Frame",{BackgroundColor3=currentShade1,BorderSizePixel=0,Name="Content",Parent={1},Position=UDim2.new(0,0,0,20),Size=UDim2.new(1,0,0,202),ZIndex=10,}},
	{7,"ScrollingFrame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,BottomImage="rbxasset://textures/ui/Scroll/scroll-middle.png",CanvasSize=UDim2.new(0,0,0,100),Name="List",Parent={6},Position=UDim2.new(0,5,0,5),ScrollBarImageColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),ScrollBarThickness=8,Size=UDim2.new(1,-10,1,-10),TopImage="rbxasset://textures/ui/Scroll/scroll-middle.png",ZIndex=10,}},
	{8,"Frame",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Name="Holder",Parent={7},Size=UDim2.new(1,0,1,0),ZIndex=10,}},
	{9,"UIListLayout",{Parent={8},SortOrder=2,}},
	{10,"Frame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BackgroundTransparency=1,BorderColor3=Color3.new(0.3137255012989,0.3137255012989,0.3137255012989),BorderSizePixel=0,ClipsDescendants=true,Name="Settings",Parent={6},Position=UDim2.new(1,0,0,0),Size=UDim2.new(0,150,1,0),ZIndex=10,}},
	{11,"Frame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),Name="Slider",Parent={10},Position=UDim2.new(0,-150,0,0),Size=UDim2.new(1,0,1,0),ZIndex=10,}},
	{12,"Frame",{BackgroundColor3=Color3.new(0.23529413342476,0.23529413342476,0.23529413342476),BorderColor3=Color3.new(0.3137255012989,0.3137255012989,0.3137255012989),BorderSizePixel=0,Name="Line",Parent={11},Size=UDim2.new(0,1,1,0),ZIndex=10,}},
	{13,"ScrollingFrame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,BottomImage="rbxasset://textures/ui/Scroll/scroll-middle.png",CanvasSize=UDim2.new(0,0,0,100),Name="List",Parent={11},Position=UDim2.new(0,0,0,25),ScrollBarImageColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),ScrollBarThickness=8,Size=UDim2.new(1,0,1,-25),TopImage="rbxasset://textures/ui/Scroll/scroll-middle.png",ZIndex=10,}},
	{14,"Frame",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Name="Holder",Parent={13},Size=UDim2.new(1,0,1,0),ZIndex=10,}},
	{15,"UIListLayout",{Parent={14},SortOrder=2,}},
	{16,"TextLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Title",Parent={11},Size=UDim2.new(1,0,0,20),Text="Event Settings",TextColor3=Color3.new(1,1,1),TextSize=14,ZIndex=10,}},
	{17,"TextButton",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),Font=3,Name="Close",BorderSizePixel=0,Parent={11},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),Text="<",TextColor3=Color3.new(1,1,1),TextSize=18,ZIndex=10,}},
	{18,"Folder",{Name="Templates",Parent={10},}},
	{19,"Frame",{BackgroundColor3=Color3.new(0.19607844948769,0.19607844948769,0.19607844948769),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),Name="Players",Parent={18},Position=UDim2.new(0,0,0,25),Size=UDim2.new(1,0,0,86),Visible=false,ZIndex=10,}},
	{20,"TextLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Title",Parent={19},Size=UDim2.new(1,0,0,20),Text="Choose Players",TextColor3=Color3.new(1,1,1),TextSize=14,ZIndex=10,}},
	{21,"TextLabel",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="Any",Parent={19},Position=UDim2.new(0,5,0,42),Size=UDim2.new(1,-10,0,20),Text="Any Player",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{22,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="Button",Parent={21},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{23,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={22},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{24,"TextLabel",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="Me",Parent={19},Position=UDim2.new(0,5,0,20),Size=UDim2.new(1,-10,0,20),Text="Me Only",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{25,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="Button",Parent={24},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{26,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={25},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{27,"TextBox",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,ClearTextOnFocus=false,Font=3,Name="Custom",Parent={19},PlaceholderColor3=Color3.new(0.47058826684952,0.47058826684952,0.47058826684952),PlaceholderText="Custom Player Set",Position=UDim2.new(0,5,0,64),Size=UDim2.new(1,-35,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{28,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="CustomButton",Parent={19},Position=UDim2.new(1,-25,0,64),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{29,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={28},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{30,"Frame",{BackgroundColor3=Color3.new(0.19607844948769,0.19607844948769,0.19607844948769),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),Name="Strings",Parent={18},Position=UDim2.new(0,0,0,25),Size=UDim2.new(1,0,0,64),Visible=false,ZIndex=10,}},
	{31,"TextLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Title",Parent={30},Size=UDim2.new(1,0,0,20),Text="Choose String",TextColor3=Color3.new(1,1,1),TextSize=14,ZIndex=10,}},
	{32,"TextLabel",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="Any",Parent={30},Position=UDim2.new(0,5,0,20),Size=UDim2.new(1,-10,0,20),Text="Any String",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{33,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="Button",Parent={32},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{34,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={33},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{54,"Frame",{BackgroundColor3=Color3.new(0.19607844948769,0.19607844948769,0.19607844948769),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),Name="Numbers",Parent={18},Position=UDim2.new(0,0,0,25),Size=UDim2.new(1,0,0,64),Visible=false,ZIndex=10,}},
	{55,"TextLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Font=3,Name="Title",Parent={54},Size=UDim2.new(1,0,0,20),Text="Choose String",TextColor3=Color3.new(1,1,1),TextSize=14,ZIndex=10,}},
	{56,"TextLabel",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="Any",Parent={54},Position=UDim2.new(0,5,0,20),Size=UDim2.new(1,-10,0,20),Text="Any Number",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{57,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="Button",Parent={56},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{58,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={57},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{59,"TextBox",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,ClearTextOnFocus=false,Font=3,Name="Custom",Parent={54},PlaceholderColor3=Color3.new(0.47058826684952,0.47058826684952,0.47058826684952),PlaceholderText="Number",Position=UDim2.new(0,5,0,42),Size=UDim2.new(1,-35,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{60,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="CustomButton",Parent={54},Position=UDim2.new(1,-25,0,42),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{61,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={60},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{35,"TextBox",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,ClearTextOnFocus=false,Font=3,Name="Custom",Parent={30},PlaceholderColor3=Color3.new(0.47058826684952,0.47058826684952,0.47058826684952),PlaceholderText="Match String",Position=UDim2.new(0,5,0,42),Size=UDim2.new(1,-35,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{36,"Frame",{BackgroundColor3=Color3.new(0.30588236451149,0.30588236451149,0.3098039329052),BorderSizePixel=0,Name="CustomButton",Parent={30},Position=UDim2.new(1,-25,0,42),Size=UDim2.new(0,20,0,20),ZIndex=10,}},
	{37,"TextButton",{BackgroundColor3=Color3.new(0.58823531866074,0.58823531866074,0.59215688705444),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="On",Parent={36},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),Text="",TextColor3=Color3.new(0,0,0),TextSize=14,ZIndex=10,}},
	{38,"Frame",{BackgroundColor3=Color3.new(0.19607844948769,0.19607844948769,0.19607844948769),BackgroundTransparency=1,BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),Name="DelayEditor",Parent={18},Position=UDim2.new(0,0,0,25),Size=UDim2.new(1,0,0,24),Visible=false,ZIndex=10,}},
	{39,"TextBox",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BorderColor3=Color3.new(0.15686275064945,0.15686275064945,0.15686275064945),BorderSizePixel=0,Font=3,Name="Secs",Parent={38},PlaceholderColor3=Color3.new(0.47058826684952,0.47058826684952,0.47058826684952),Position=UDim2.new(0,60,0,2),Size=UDim2.new(1,-65,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{40,"TextLabel",{BackgroundColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),BackgroundTransparency=1,BorderSizePixel=0,Font=3,Name="Label",Parent={39},Position=UDim2.new(0,-55,0,0),Size=UDim2.new(1,0,1,0),Text="Delay (s):",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{41,"Frame",{BackgroundColor3=currentShade1,BorderSizePixel=0,ClipsDescendants=true,Name="EventTemplate",Parent={6},Size=UDim2.new(1,0,0,20),Visible=false,ZIndex=10,}},
	{42,"TextButton",{BackgroundColor3=currentText1,BackgroundTransparency=1,Font=3,Name="Expand",Parent={41},Size=UDim2.new(0,20,0,20),Text=">",TextColor3=Color3.new(1,1,1),TextSize=18,ZIndex=10,}},
	{43,"TextLabel",{BackgroundColor3=currentText1,BackgroundTransparency=1,Font=3,Name="EventName",Parent={41},Position=UDim2.new(0,25,0,0),Size=UDim2.new(1,-25,0,20),Text="OnSpawn",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{44,"Frame",{BackgroundColor3=Color3.new(0.19607844948769,0.19607844948769,0.19607844948769),BorderSizePixel=0,BackgroundTransparency=1,ClipsDescendants=true,Name="Cmds",Parent={41},Position=UDim2.new(0,0,0,20),Size=UDim2.new(1,0,1,-20),ZIndex=10,}},
	{45,"Frame",{BackgroundColor3=Color3.new(0.14117647707462,0.14117647707462,0.14509804546833),BorderColor3=Color3.new(0.1803921610117,0.1803921610117,0.1843137294054),Name="Add",Parent={44},Position=UDim2.new(0,0,1,-20),Size=UDim2.new(1,0,0,20),ZIndex=10,}},
	{46,"TextBox",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,ClearTextOnFocus=false,Font=3,Parent={45},PlaceholderColor3=Color3.new(0.7843137383461,0.7843137383461,0.7843137383461),PlaceholderText="Add new command",Position=UDim2.new(0,5,0,0),Size=UDim2.new(1,-10,1,0),Text="",TextColor3=Color3.new(1,1,1),TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{47,"Frame",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Name="Holder",Parent={44},Size=UDim2.new(1,0,1,-20),ZIndex=10,}},
	{48,"UIListLayout",{Parent={47},SortOrder=2,}},
	{49,"Frame",{currentShade1,BorderSizePixel=0,ClipsDescendants=true,Name="CmdTemplate",Parent={6},Size=UDim2.new(1,0,0,20),Visible=false,ZIndex=10,}},
	{50,"TextBox",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,ClearTextOnFocus=false,Font=3,Parent={49},PlaceholderColor3=Color3.new(1,1,1),Position=UDim2.new(0,5,0,0),Size=UDim2.new(1,-45,0,20),Text="a\\b\\c\\d",TextColor3=currentText1,TextSize=14,TextXAlignment=0,ZIndex=10,}},
	{51,"TextButton",{BackgroundColor3=currentShade1,BorderSizePixel=0,Font=3,Name="Delete",Parent={49},Position=UDim2.new(1,-20,0,0),Size=UDim2.new(0,20,0,20),Text="X",TextColor3=Color3.new(1,1,1),TextSize=18,ZIndex=10,}},
	{52,"TextButton",{BackgroundColor3=currentShade1,BorderSizePixel=0,Font=3,Name="Settings",Parent={49},Position=UDim2.new(1,-40,0,0),Size=UDim2.new(0,20,0,20),Text="",TextColor3=Color3.new(1,1,1),TextSize=18,ZIndex=10,}},
	{53,"ImageLabel",{BackgroundColor3=Color3.new(1,1,1),BackgroundTransparency=1,Image="rbxassetid://1204397029",Parent={52},Position=UDim2.new(0,2,0,2),Size=UDim2.new(0,16,0,16),ZIndex=10,}},
})
main.Name = randomString()
local mainFrame = main:WaitForChild("Content")
local eventList = mainFrame:WaitForChild("List")
local eventListHolder = eventList:WaitForChild("Holder")
local cmdTemplate = mainFrame:WaitForChild("CmdTemplate")
local eventTemplate = mainFrame:WaitForChild("EventTemplate")
local settingsFrame = mainFrame:WaitForChild("Settings"):WaitForChild("Slider")
local settingsTemplates = mainFrame.Settings:WaitForChild("Templates")
local settingsList = settingsFrame:WaitForChild("List"):WaitForChild("Holder")
table.insert(shade2,main.TopBar) table.insert(shade1,mainFrame) table.insert(shade2,eventTemplate)
table.insert(text1,eventTemplate.EventName) table.insert(shade1,eventTemplate.Cmds.Add) table.insert(shade1,cmdTemplate)
table.insert(text1,cmdTemplate.TextBox) table.insert(shade2,cmdTemplate.Delete) table.insert(shade2,cmdTemplate.Settings)
table.insert(scroll,mainFrame.List) table.insert(shade1,settingsFrame) table.insert(shade2,settingsFrame.Line)
table.insert(shade2,settingsFrame.Close) table.insert(scroll,settingsFrame.List) table.insert(shade2,settingsTemplates.DelayEditor.Secs)
table.insert(text1,settingsTemplates.DelayEditor.Secs) table.insert(text1,settingsTemplates.DelayEditor.Secs.Label) table.insert(text1,settingsTemplates.Players.Title)
table.insert(shade3,settingsTemplates.Players.CustomButton) table.insert(shade2,settingsTemplates.Players.Custom) table.insert(text1,settingsTemplates.Players.Custom)
table.insert(shade3,settingsTemplates.Players.Any.Button) table.insert(shade3,settingsTemplates.Players.Me.Button) table.insert(text1,settingsTemplates.Players.Any)
table.insert(text1,settingsTemplates.Players.Me) table.insert(text1,settingsTemplates.Strings.Title) table.insert(text1,settingsTemplates.Strings.Any)
table.insert(shade3,settingsTemplates.Strings.Any.Button) table.insert(shade3,settingsTemplates.Strings.CustomButton) table.insert(text1,settingsTemplates.Strings.Custom)
table.insert(shade2,settingsTemplates.Strings.Custom)
table.insert(text1,settingsTemplates.Players.Me) table.insert(text1,settingsTemplates.Numbers.Title) table.insert(text1,settingsTemplates.Numbers.Any)
table.insert(shade3,settingsTemplates.Numbers.Any.Button) table.insert(shade3,settingsTemplates.Numbers.CustomButton) table.insert(text1,settingsTemplates.Numbers.Custom)
table.insert(shade2,settingsTemplates.Numbers.Custom)

local tweenInf = TweenInfo.new(0.25,Enum.EasingStyle.Quart,Enum.EasingDirection.Out)

local currentlyEditingCmd = nil

settingsFrame:WaitForChild("Close").MouseButton1Click:Connect(function()
	settingsFrame:TweenPosition(UDim2.new(0,-150,0,0),Enum.EasingDirection.Out,Enum.EasingStyle.Quart,0.25,true)
end)

local function resizeList()
	local size = 0

	for i,v in pairs(eventListHolder:GetChildren()) do
		if v.Name == "EventTemplate" then
			size = size + 20
			if v.Expand.Rotation == 90 then
				size = size + 20*(1+(#events[v.EventName:GetAttribute("RawName")].commands or 0))
			end
		end
	end

	TweenService:Create(eventList,tweenInf,{CanvasSize = UDim2.new(0,0,0,size)}):Play()

	if size > eventList.AbsoluteSize.Y then
		eventListHolder.Size = UDim2.new(1,-8,1,0)
	else
		eventListHolder.Size = UDim2.new(1,0,1,0)
	end
end

local function resizeSettingsList()
	local size = 0

	for i,v in pairs(settingsList:GetChildren()) do
		if v:IsA("Frame") then
			size = size + v.AbsoluteSize.Y
		end
	end

	settingsList.Parent.CanvasSize = UDim2.new(0,0,0,size)

	if size > settingsList.Parent.AbsoluteSize.Y then
		settingsList.Size = UDim2.new(1,-8,1,0)
	else
		settingsList.Size = UDim2.new(1,0,1,0)
	end
end

local function setupCheckbox(button,callback)
	local enabled = button.On.BackgroundTransparency == 0

	local function update()
		button.On.BackgroundTransparency = (enabled and 0 or 1)
	end

	button.On.MouseButton1Click:Connect(function()
		enabled = not enabled
		update()
		if callback then callback(enabled) end
	end)

	return {
		Toggle = function(nocall) enabled = not enabled update() if not nocall and callback then callback(enabled) end end,
		Enable = function(nocall) if enabled then return end enabled = true update()if not nocall and callback then callback(enabled) end end,
		Disable = function(nocall) if not enabled then return end enabled = false update()if not nocall and callback then callback(enabled) end end,
		IsEnabled = function() return enabled end
	}
end

local function openSettingsEditor(event,cmd)
	currentlyEditingCmd = cmd

	for i,v in pairs(settingsList:GetChildren()) do if v:IsA("Frame") then v:Destroy() end end

	local delayEditor = settingsTemplates.DelayEditor:Clone()
	delayEditor.Secs.FocusLost:Connect(function()
		cmd[3] = tonumber(delayEditor.Secs.Text) or 0
		delayEditor.Secs.Text = cmd[3]
		if onEdited then onEdited() end
	end)
	delayEditor.Secs.Text = cmd[3]
	delayEditor.Visible = true
	table.insert(shade2,delayEditor.Secs)
	table.insert(text1,delayEditor.Secs)
	table.insert(text1,delayEditor.Secs.Label)
	delayEditor.Parent = settingsList

	for i,v in pairs(event.sets) do
		if v.Type == "Player" then
			local template = settingsTemplates.Players:Clone()
			template.Title.Text = v.Name or "Player"

			local me,any,custom

			me = setupCheckbox(template.Me.Button,function(on)
				if not on then return end
				any.Disable()
				custom.Disable()
				cmd[2][i] = 0
				if onEdited then onEdited() end
			end)

			any = setupCheckbox(template.Any.Button,function(on)
				if not on then return end
				me.Disable()
				custom.Disable()
				cmd[2][i] = 1
				if onEdited then onEdited() end
			end)

			local customTextBox = template.Custom
			custom = setupCheckbox(template.CustomButton,function(on)
				if not on then return end
				me.Disable()
				any.Disable()
				cmd[2][i] = customTextBox.Text
				if onEdited then onEdited() end
			end)

			ViewportTextBox.convert(customTextBox)
			customTextBox.FocusLost:Connect(function()
				if custom:IsEnabled() then
					cmd[2][i] = customTextBox.Text
					if onEdited then onEdited() end
				end
			end)

			local cVal = cmd[2][i]
			if cVal == 0 then
				me:Enable()
			elseif cVal == 1 then
				any:Enable()
			else
				custom:Enable()
				customTextBox.Text = cVal
			end

			template.Visible = true
			table.insert(text1,template.Title)
			table.insert(shade3,template.CustomButton)
			table.insert(shade3,template.Any.Button)
			table.insert(shade3,template.Me.Button)
			table.insert(text1,template.Any)
			table.insert(text1,template.Me)
			template.Parent = settingsList
		elseif v.Type == "String" then
			local template = settingsTemplates.Strings:Clone()
			template.Title.Text = v.Name or "String"

			local any,custom

			any = setupCheckbox(template.Any.Button,function(on)
				if not on then return end
				custom.Disable()
				cmd[2][i] = 0
				if onEdited then onEdited() end
			end)

			local customTextBox = template.Custom
			custom = setupCheckbox(template.CustomButton,function(on)
				if not on then return end
				any.Disable()
				cmd[2][i] = customTextBox.Text
				if onEdited then onEdited() end
			end)

			ViewportTextBox.convert(customTextBox)
			customTextBox.FocusLost:Connect(function()
				if custom:IsEnabled() then
					cmd[2][i] = customTextBox.Text
					if onEdited then onEdited() end
				end
			end)

			local cVal = cmd[2][i]
			if cVal == 0 then
				any:Enable()
			else
				custom:Enable()
				customTextBox.Text = cVal
			end

			template.Visible = true
			table.insert(text1,template.Title)
			table.insert(text1,template.Any)
			table.insert(shade3,template.Any.Button)
			table.insert(shade3,template.CustomButton)
			template.Parent = settingsList
		elseif v.Type == "Number" then
			local template = settingsTemplates.Numbers:Clone()
			template.Title.Text = v.Name or "Number"

			local any,custom

			any = setupCheckbox(template.Any.Button,function(on)
				if not on then return end
				custom.Disable()
				cmd[2][i] = 0
				if onEdited then onEdited() end
			end)

			local customTextBox = template.Custom
			custom = setupCheckbox(template.CustomButton,function(on)
				if not on then return end
				any.Disable()
				cmd[2][i] = customTextBox.Text
				if onEdited then onEdited() end
			end)

			ViewportTextBox.convert(customTextBox)
			customTextBox.FocusLost:Connect(function()
				cmd[2][i] = tonumber(customTextBox.Text) or 0
				customTextBox.Text = cmd[2][i]
				if custom:IsEnabled() then
					if onEdited then onEdited() end
				end
			end)

			local cVal = cmd[2][i]
			if cVal == 0 then
				any:Enable()
			else
				custom:Enable()
				customTextBox.Text = cVal
			end

			template.Visible = true
			table.insert(text1,template.Title)
			table.insert(text1,template.Any)
			table.insert(shade3,template.Any.Button)
			table.insert(shade3,template.CustomButton)
			template.Parent = settingsList
		end
	end
	resizeSettingsList()
	settingsFrame:TweenPosition(UDim2.new(0,0,0,0),Enum.EasingDirection.Out,Enum.EasingStyle.Quart,0.25,true)
end

local function defaultSettings(ev)
	local res = {}

	for i,v in pairs(ev.sets) do
		if v.Type == "Player" then
			res[#res+1] = v.Default or 0
		elseif v.Type == "String" then
			res[#res+1] = v.Default or 0
		elseif v.Type == "Number" then
			res[#res+1] = v.Default or 0
		end
	end

	return res
end

local function refreshList()
	for i,v in pairs(eventListHolder:GetChildren()) do if v:IsA("Frame") then v:Destroy() end end

	for name,event in pairs(events) do
		local eventF = eventTemplate:Clone()
		eventF.EventName.Text = name
		eventF.Visible = true
		eventF.EventName:SetAttribute("RawName", name)
		table.insert(shade2,eventF)
		table.insert(text1,eventF.EventName)
		table.insert(shade1,eventF.Cmds.Add)

		local expanded = false
		eventF.Expand.MouseButton1Down:Connect(function()
			expanded = not expanded
			eventF:TweenSize(UDim2.new(1,0,0,20 + (expanded and 20*#eventF.Cmds.Holder:GetChildren() or 0)),Enum.EasingDirection.Out,Enum.EasingStyle.Quart,0.25,true)
			eventF.Expand.Rotation = expanded and 90 or 0
			resizeList()
		end)

		local function refreshCommands()
			for i,v in pairs(eventF.Cmds.Holder:GetChildren()) do
				if v.Name == "CmdTemplate" then
					v:Destroy()
				end
			end

			eventF.EventName.Text = name..(#event.commands > 0 and " ("..#event.commands..")" or "")

			for i,cmd in pairs(event.commands) do
				local cmdF = cmdTemplate:Clone()
				local cmdTextBox = cmdF.TextBox
				ViewportTextBox.convert(cmdTextBox)
				cmdTextBox.Text = cmd[1]
				cmdF.Visible = true
				table.insert(shade1,cmdF)
				table.insert(shade2,cmdF.Delete)
				table.insert(shade2,cmdF.Settings)

				cmdTextBox.FocusLost:Connect(function()
					event.commands[i] = {cmdTextBox.Text,cmd[2],cmd[3]}
					if onEdited then onEdited() end
				end)

				cmdF.Settings.MouseButton1Click:Connect(function()
					openSettingsEditor(event,cmd)
				end)

				cmdF.Delete.MouseButton1Click:Connect(function()
					table.remove(event.commands,i)
					refreshCommands()
					resizeList()

					if currentlyEditingCmd == cmd then
						settingsFrame:TweenPosition(UDim2.new(0,-150,0,0),Enum.EasingDirection.Out,Enum.EasingStyle.Quart,0.25,true)
					end
					if onEdited then onEdited() end
				end)

				cmdF.Parent = eventF.Cmds.Holder
			end

			eventF:TweenSize(UDim2.new(1,0,0,20 + (expanded and 20*#eventF.Cmds.Holder:GetChildren() or 0)),Enum.EasingDirection.Out,Enum.EasingStyle.Quart,0.25,true)
		end

		local newBox = eventF.Cmds.Add.TextBox
		ViewportTextBox.convert(newBox)
		newBox.FocusLost:Connect(function(enter)
			if enter then
				event.commands[#event.commands+1] = {newBox.Text,defaultSettings(event),0}
				newBox.Text = ""

				refreshCommands()
				resizeList()
				if onEdited then onEdited() end
			end
		end)

		--eventF:GetPropertyChangedSignal("AbsoluteSize"):Connect(resizeList)

		eventF.Parent = eventListHolder

		refreshCommands()
	end

	resizeList()
end

local function saveData()
	local result = {}
	for i,v in pairs(events) do
		result[i] = v.commands
	end
	return HttpService:JSONEncode(result)
end

local function loadData(str)
	local data = HttpService:JSONDecode(str)
	for i,v in pairs(data) do
		if events[i] then
			events[i].commands = v
		end
	end
end

local function addCmd(event,data)
	table.insert(events[event].commands,data)
end

local function setOnEdited(f)
	if type(f) == "function" then
		onEdited = f
	end
end

main.TopBar.Close.MouseButton1Click:Connect(function()
	main:TweenPosition(UDim2.new(0.5,-175,0,-500), "InOut", "Quart", 0.5, true, nil)
end)
dragGUI(main)
main.Parent = PARENT

return {
	RegisterEvent = registerEvent,
	FireEvent = fireEvent,
	Refresh = refreshList,
	SaveData = saveData,
	LoadData = loadData,
	AddCmd = addCmd,
	Frame = main,
	SetOnEdited = setOnEdited
}


FileError.Name = randomString()
FileError.Parent = PARENT
FileError.Active = true
FileError.BackgroundTransparency = 1
FileError.Position = UDim2.new(0.5, -180, 0, 290)
FileError.Size = UDim2.new(0, 360, 0, 20)
FileError.ZIndex = 10

background.Name = "background"
background.Parent = FileError
background.Active = true
background.BackgroundColor3 = Color3.fromRGB(36, 36, 37)
background.BorderSizePixel = 0
background.Position = UDim2.new(0, 0, 0, 20)
background.Size = UDim2.new(0, 360, 0, 205)
background.ZIndex = 10

Directions.Name = "Directions"
Directions.Parent = background
Directions.BackgroundTransparency = 1
Directions.BorderSizePixel = 0
Directions.Position = UDim2.new(0, 10, 0, 10)
Directions.Size = UDim2.new(0, 340, 0, 185)
Directions.Font = Enum.Font.SourceSans
Directions.TextSize = 14
Directions.Text = text
Directions.TextColor3 = Color3.new(1, 1, 1)
Directions.TextWrapped = true
Directions.TextXAlignment = Enum.TextXAlignment.Left
Directions.TextYAlignment = Enum.TextYAlignment.Top
Directions.ZIndex = 10

shadow.Name = "shadow"
shadow.Parent = FileError
shadow.BackgroundColor3 = Color3.fromRGB(46, 46, 47)
shadow.BorderSizePixel = 0
shadow.Size = UDim2.new(0, 360, 0, 20)
shadow.ZIndex = 10

PopupText.Name = "PopupText"
PopupText.Parent = shadow
PopupText.BackgroundTransparency = 1
PopupText.Size = UDim2.new(1, 0, 0.95, 0)
PopupText.ZIndex = 10
PopupText.Font = Enum.Font.SourceSans
PopupText.TextSize = 14
PopupText.Text = "File Error"
PopupText.TextColor3 = Color3.new(1, 1, 1)
PopupText.TextWrapped = true

Exit.Name = "Exit"
Exit.Parent = shadow
Exit.BackgroundTransparency = 1
Exit.Position = UDim2.new(1, -20, 0, 0)
Exit.Size = UDim2.new(0, 20, 0, 20)
Exit.Text = ""
Exit.ZIndex = 10

ExitImage.Parent = Exit
ExitImage.BackgroundColor3 = Color3.new(1, 1, 1)
ExitImage.BackgroundTransparency = 1
ExitImage.Position = UDim2.new(0, 5, 0, 5)
ExitImage.Size = UDim2.new(0, 10, 0, 10)
ExitImage.Image = "rbxassetid://5054663650"
ExitImage.ZIndex = 10

Exit.MouseButton1Click:Connect(function()
    FileError:Destroy()
end)
