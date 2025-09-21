this is made all by serverscript.

https://www.roblox.com/games/100267451453127/preview 

when joined look up.

serverscript code. (for proof)

------

local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local Debris = game:GetService("Debris")
local Lighting = game:GetService("Lighting")

local VOXEL_SIZE = 4
local GRID_SIZE = 20
local PHI = 1.618033988749895
local TAU = math.pi * 2

local VoxelEffects = {}
VoxelEffects.__index = VoxelEffects

function VoxelEffects.new(pos, size)
	local self = setmetatable({}, VoxelEffects)
	self.position = pos or Vector3.new(0, 50, 0)
	self.size = size or GRID_SIZE
	self.voxels = {}
	self.activeEffects = {}
	self.folder = Instance.new("Folder")
	self.folder.Name = "VoxelMatrix"
	self.folder.Parent = Workspace
	self.materialCycle = {
		Enum.Material.Neon,
		Enum.Material.ForceField,
		Enum.Material.Glass,
		Enum.Material.DiamondPlate,
		Enum.Material.Granite
	}
	return self
end

function VoxelEffects:createVoxel(x, y, z, color, material, transparency)
	local part = Instance.new("Part")
	part.Name = "Voxel_" .. x .. "_" .. y .. "_" .. z
	part.Size = Vector3.new(VOXEL_SIZE, VOXEL_SIZE, VOXEL_SIZE)
	part.Position = self.position + Vector3.new(x * VOXEL_SIZE, y * VOXEL_SIZE, z * VOXEL_SIZE)
	part.Color = color or Color3.fromRGB(255, 255, 255)
	part.Material = material or Enum.Material.Neon
	part.Transparency = transparency or 0
	part.Anchored = true
	part.CanCollide = false
	part.CFrame = part.CFrame * CFrame.Angles(math.rad(math.random(-5, 5)), math.rad(math.random(-5, 5)), math.rad(math.random(-5, 5)))
	part.Parent = self.folder

	local light = Instance.new("PointLight")
	light.Brightness = 0.5
	light.Range = VOXEL_SIZE * 2
	light.Color = color or Color3.new(1, 1, 1)
	light.Parent = part

	if not self.voxels[x] then 
		self.voxels[x] = {} 
	end
	if not self.voxels[x][y] then 
		self.voxels[x][y] = {} 
	end
	self.voxels[x][y][z] = part

	return part
end

function VoxelEffects:removeVoxel(x, y, z)
	if self.voxels[x] and self.voxels[x][y] and self.voxels[x][y][z] then
		self.voxels[x][y][z]:Destroy()
		self.voxels[x][y][z] = nil
	end
end

function VoxelEffects:clear()
	for _, child in pairs(self.folder:GetChildren()) do
		child:Destroy()
	end
	self.voxels = {}
end

function VoxelEffects:stopAllEffects()
	for _, connection in pairs(self.activeEffects) do
		connection:Disconnect()
	end
	self.activeEffects = {}
end

function VoxelEffects:quantumSphere(maxRadius, duration, layers)
	self:clear()
	local center = Vector3.new(self.size/2, self.size/2, self.size/2)

	local startTime = tick()
	local connection
	connection = RunService.Heartbeat:Connect(function()
		local elapsed = tick() - startTime
		local progress = math.min(elapsed / duration, 1)

		self:clear()

		for layer = 1, layers do
			local layerRadius = (maxRadius / layers) * layer * progress
			local layerOffset = math.sin(elapsed * 2 + layer) * 2

			for theta = 0, math.pi, math.pi / 10 do
				for phi = 0, TAU, TAU / 20 do
					local x = math.floor(center.X + (layerRadius + layerOffset) * math.sin(theta) * math.cos(phi))
					local y = math.floor(center.Y + (layerRadius + layerOffset) * math.sin(theta) * math.sin(phi))
					local z = math.floor(center.Z + (layerRadius + layerOffset) * math.cos(theta))

					if x >= 0 and x < self.size and y >= 0 and y < self.size and z >= 0 and z < self.size then
						local hue = ((layer / layers) + elapsed * 0.1) % 1
						local color = Color3.fromHSV(hue, 0.9, 1)
						local transparency = 0.3 + (layer / layers) * 0.4
						self:createVoxel(x, y, z, color, Enum.Material.ForceField, transparency)
					end
				end
			end
		end

		if progress >= 1 then
			connection:Disconnect()
		end
	end)

	table.insert(self.activeEffects, connection)
end

function VoxelEffects:fractalTree(depth, branchLength, angleVariation)
	self:clear()
	local center = Vector3.new(self.size/2, 0, self.size/2)

	local function growBranch(pos, direction, currentDepth, currentLength)
		if currentDepth <= 0 or currentLength < 1 then
			return
		end

		local endPos = pos + direction * currentLength

		for t = 0, 1, 0.2 do
			local interpolated = pos:Lerp(endPos, t)
			local x, y, z = math.floor(interpolated.X), math.floor(interpolated.Y), math.floor(interpolated.Z)

			if x >= 0 and x < self.size and y >= 0 and y < self.size and z >= 0 and z < self.size then
				local depthRatio = currentDepth / depth
				local color = Color3.fromHSV(0.1 + (1 - depthRatio) * 0.2, 0.8, depthRatio)
				self:createVoxel(x, y, z, color, Enum.Material.Wood)
			end
		end

		local branches = math.random(2, 4)
		for i = 1, branches do
			local angleX = math.rad(math.random(-angleVariation, angleVariation))
			local angleY = math.rad(math.random(-180, 180))
			local angleZ = math.rad(math.random(-angleVariation, angleVariation))

			local newDirection = CFrame.new(Vector3.zero, direction) * CFrame.Angles(angleX, angleY, angleZ)
			newDirection = newDirection.LookVector

			growBranch(endPos, newDirection, currentDepth - 1, currentLength * 0.7)
		end
	end

	growBranch(center, Vector3.new(0, 1, 0), depth, branchLength)
end

function VoxelEffects:helixDNA(strands, rotationSpeed, duration)
	self:clear()
	local centerX = self.size / 2
	local centerZ = self.size / 2
	local radius = self.size / 4

	local startTime = tick()
	local connection
	connection = RunService.Heartbeat:Connect(function()
		local elapsed = tick() - startTime
		if elapsed > duration then
			connection:Disconnect()
			return
		end

		self:clear()

		for strand = 1, strands do
			local strandOffset = (TAU / strands) * strand

			for y = 0, self.size - 1 do
				local angle = (y / self.size) * TAU * 2 + elapsed * rotationSpeed + strandOffset

				local x1 = math.floor(centerX + math.cos(angle) * radius)
				local z1 = math.floor(centerZ + math.sin(angle) * radius)
				local x2 = math.floor(centerX - math.cos(angle) * radius)
				local z2 = math.floor(centerZ - math.sin(angle) * radius)

				local hue = ((y / self.size) + elapsed * 0.2) % 1
				local color1 = Color3.fromHSV(hue, 1, 1)
				local color2 = Color3.fromHSV((hue + 0.5) % 1, 1, 1)

				if x1 >= 0 and x1 < self.size and z1 >= 0 and z1 < self.size then
					self:createVoxel(x1, y, z1, color1, Enum.Material.Neon)
				end

				if x2 >= 0 and x2 < self.size and z2 >= 0 and z2 < self.size then
					self:createVoxel(x2, y, z2, color2, Enum.Material.Neon)
				end

				if y % 3 == 0 then
					for t = 0.2, 0.8, 0.2 do
						local bridgeX = math.floor(x1 + (x2 - x1) * t)
						local bridgeZ = math.floor(z1 + (z2 - z1) * t)

						if bridgeX >= 0 and bridgeX < self.size and bridgeZ >= 0 and bridgeZ < self.size then
							self:createVoxel(bridgeX, y, bridgeZ, Color3.fromRGB(200, 200, 200), Enum.Material.Glass, 0.5)
						end
					end
				end
			end
		end
	end)

	table.insert(self.activeEffects, connection)
end

function VoxelEffects:vortexTunnel(innerRadius, outerRadius, twistSpeed, duration)
	self:clear()
	local center = Vector3.new(self.size/2, self.size/2, self.size/2)

	local startTime = tick()
	local connection
	connection = RunService.Heartbeat:Connect(function()
		local elapsed = tick() - startTime
		if elapsed > duration then
			connection:Disconnect()
			return
		end

		self:clear()

		for y = 0, self.size - 1 do
			local yProgress = y / self.size
			local twist = elapsed * twistSpeed + yProgress * TAU

			for angle = 0, TAU, TAU / 30 do
				local currentAngle = angle + twist

				for r = innerRadius, outerRadius, 1 do
					local x = math.floor(center.X + math.cos(currentAngle) * r)
					local z = math.floor(center.Z + math.sin(currentAngle) * r)

					if x >= 0 and x < self.size and z >= 0 and z < self.size then
						local radiusRatio = (r - innerRadius) / (outerRadius - innerRadius)
						local hue = (yProgress + radiusRatio + elapsed * 0.1) % 1
						local brightness = 0.5 + math.sin(elapsed * 3 + angle) * 0.5
						local color = Color3.fromHSV(hue, 0.8, brightness)

						self:createVoxel(x, y, z, color, Enum.Material.ForceField, radiusRatio * 0.5)
					end
				end
			end
		end
	end)

	table.insert(self.activeEffects, connection)
end

function VoxelEffects:matrixRain(columns, dropSpeed, duration)
	self:clear()

	local drops = {}
	for col = 1, columns do
		local x = math.random(0, self.size - 1)
		local z = math.random(0, self.size - 1)

		drops[col] = {
			x = x,
			z = z,
			y = math.random(self.size - 1, self.size * 1.5),
			speed = dropSpeed * (0.5 + math.random()),
			length = math.random(5, 10)
		}
	end

	local startTime = tick()
	local connection
	connection = RunService.Heartbeat:Connect(function()
		local elapsed = tick() - startTime
		if elapsed > duration then
			connection:Disconnect()
			return
		end

		self:clear()

		for col, drop in ipairs(drops) do
			drop.y = drop.y - drop.speed

			for i = 0, drop.length - 1 do
				local y = math.floor(drop.y + i)

				if y >= 0 and y < self.size then
					local brightness = 1 - (i / drop.length)
					local transparency = i / drop.length * 0.8
					local color = Color3.fromRGB(0, 255 * brightness, 100 * brightness)

					self:createVoxel(drop.x, y, drop.z, color, Enum.Material.Neon, transparency)
				end
			end

			if drop.y + drop.length < 0 then
				drop.y = self.size + math.random(0, 10)
				drop.x = math.random(0, self.size - 1)
				drop.z = math.random(0, self.size - 1)
				drop.length = math.random(5, 10)
			end
		end
	end)

	table.insert(self.activeEffects, connection)
end

function VoxelEffects:goldenSpiral(growth, rotations, buildSpeed)
	self:clear()
	local center = Vector3.new(self.size/2, self.size/2, self.size/2)

	local maxPoints = rotations * 100
	local currentPoint = 0

	local startTime = tick()
	local connection
	connection = RunService.Heartbeat:Connect(function()
		local elapsed = tick() - startTime
		currentPoint = math.min(elapsed * buildSpeed, maxPoints)

		for i = math.max(0, currentPoint - 50), currentPoint do
			local angle = i * PHI * TAU / 100
			local radius = growth * math.sqrt(i)
			local height = (i / maxPoints) * self.size

			local x = math.floor(center.X + math.cos(angle) * radius)
			local z = math.floor(center.Z + math.sin(angle) * radius)
			local y = math.floor(height)

			if x >= 0 and x < self.size and y >= 0 and y < self.size and z >= 0 and z < self.size then
				if not (self.voxels[x] and self.voxels[x][y] and self.voxels[x][y][z]) then
					local hue = (i / maxPoints) % 1
					local color = Color3.fromHSV(hue, 0.8, 1)
					self:createVoxel(x, y, z, color, Enum.Material.Neon)
				end
			end
		end

		if currentPoint >= maxPoints then
			connection:Disconnect()
		end
	end)

	table.insert(self.activeEffects, connection)
end

function VoxelEffects:crystallization(seedCount, growthRate, maxSize)
	self:clear()

	local crystals = {}
	for i = 1, seedCount do
		crystals[i] = {
			origin = Vector3.new(
				math.random(0, self.size - 1),
				math.random(0, self.size - 1),
				math.random(0, self.size - 1)
			),
			color = Color3.fromHSV(math.random(), 0.6, 1),
			size = 0,
			growthPattern = math.random() * TAU
		}
	end

	local startTime = tick()
	local connection
	connection = RunService.Heartbeat:Connect(function()
		local elapsed = tick() - startTime

		for _, crystal in ipairs(crystals) do
			crystal.size = math.min(crystal.size + growthRate, maxSize)
			local currentSize = crystal.size

			for dx = -currentSize, currentSize do
				for dy = -currentSize, currentSize do
					for dz = -currentSize, currentSize do
						local distance = math.sqrt(dx*dx + dy*dy + dz*dz)

						if distance <= currentSize and distance > currentSize - growthRate then
							local x = math.floor(crystal.origin.X + dx)
							local y = math.floor(crystal.origin.Y + dy)
							local z = math.floor(crystal.origin.Z + dz)

							if x >= 0 and x < self.size and y >= 0 and y < self.size and z >= 0 and z < self.size then
								if not (self.voxels[x] and self.voxels[x][y] and self.voxels[x][y][z]) then
									local transparency = (distance / currentSize) * 0.7
									self:createVoxel(x, y, z, crystal.color, Enum.Material.Ice, transparency)
								end
							end
						end
					end
				end
			end
		end

		local allMaxSize = true
		for _, crystal in ipairs(crystals) do
			if crystal.size < maxSize then
				allMaxSize = false
				break
			end
		end

		if allMaxSize then
			connection:Disconnect()
		end
	end)

	table.insert(self.activeEffects, connection)
end

function VoxelEffects:nebulaClouds(cloudCount, particleDensity, duration)
	self:clear()

	local clouds = {}
	for i = 1, cloudCount do
		clouds[i] = {
			center = Vector3.new(
				math.random(0, self.size - 1),
				math.random(0, self.size - 1),
				math.random(0, self.size - 1)
			),
			radius = math.random(3, 6),
			color1 = Color3.fromHSV(math.random(), 0.8, 1),
			color2 = Color3.fromHSV(math.random(), 0.6, 0.8),
			drift = Vector3.new(
				(math.random() - 0.5) * 0.2,
				(math.random() - 0.5) * 0.2,
				(math.random() - 0.5) * 0.2
			)
		}
	end

	local startTime = tick()
	local connection
	connection = RunService.Heartbeat:Connect(function()
		local elapsed = tick() - startTime
		if elapsed > duration then
			connection:Disconnect()
			return
		end

		self:clear()

		for _, cloud in ipairs(clouds) do
			cloud.center = cloud.center + cloud.drift

			for i = 1, particleDensity do
				local theta = math.random() * TAU
				local phi = math.random() * math.pi
				local r = math.random() * cloud.radius

				local x = math.floor(cloud.center.X + r * math.sin(phi) * math.cos(theta))
				local y = math.floor(cloud.center.Y + r * math.sin(phi) * math.sin(theta))
				local z = math.floor(cloud.center.Z + r * math.cos(phi))

				if x >= 0 and x < self.size and y >= 0 and y < self.size and z >= 0 and z < self.size then
					local lerpValue = r / cloud.radius
					local color = cloud.color1:Lerp(cloud.color2, lerpValue)
					local transparency = 0.3 + lerpValue * 0.6

					self:createVoxel(x, y, z, color, Enum.Material.ForceField, transparency)
				end
			end

			if cloud.center.X < -cloud.radius then cloud.center = Vector3.new(self.size + cloud.radius, cloud.center.Y, cloud.center.Z) end
			if cloud.center.X > self.size + cloud.radius then cloud.center = Vector3.new(-cloud.radius, cloud.center.Y, cloud.center.Z) end
			if cloud.center.Y < -cloud.radius then cloud.center = Vector3.new(cloud.center.X, self.size + cloud.radius, cloud.center.Z) end
			if cloud.center.Y > self.size + cloud.radius then cloud.center = Vector3.new(cloud.center.X, -cloud.radius, cloud.center.Z) end
			if cloud.center.Z < -cloud.radius then cloud.center = Vector3.new(cloud.center.X, cloud.center.Y, self.size + cloud.radius) end
			if cloud.center.Z > self.size + cloud.radius then cloud.center = Vector3.new(cloud.center.X, cloud.center.Y, -cloud.radius) end
		end
	end)

	table.insert(self.activeEffects, connection)
end

local function demonstrateEffects()
	local voxelSystem = VoxelEffects.new(Vector3.new(0, 50, 0), 24)

	print("Initializing Quantum Sphere...")
	voxelSystem:quantumSphere(10, 4, 3)
	wait(5)

	print("Generating Fractal Tree...")
	voxelSystem:fractalTree(5, 8, 30)
	wait(4)

	print("Activating DNA Helix...")
	voxelSystem:helixDNA(2, 2, 5)
	wait(6)

	print("Creating Vortex Tunnel...")
	voxelSystem:vortexTunnel(3, 8, 3, 5)
	wait(6)

	print("Starting Matrix Rain...")
	voxelSystem:matrixRain(15, 3, 6)
	wait(7)

	print("Building Golden Spiral...")
	voxelSystem:goldenSpiral(0.5, 5, 100)
	wait(5)

	print("Crystal Formation...")
	voxelSystem:crystallization(3, 0.3, 6)
	wait(5)

	print("Nebula Clouds...")
	voxelSystem:nebulaClouds(4, 30, 8)
	wait(9)

	voxelSystem:stopAllEffects()
	voxelSystem.folder:Destroy()
end

demonstrateEffects()


------------
