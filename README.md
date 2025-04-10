-- AdvancedAI.lua
-- By Aura_Mage11

local RunService = game:GetService("RunService")

local AI = {}
AI.__index = AI

function AI.new(npc, patrolPoints)
	local self = setmetatable({}, AI)
	self.NPC = npc
	self.State = "Idle"
	self.Target = nil
	self.AttackRange = 6
	self.SightRange = 30
	self.PatrolPoints = patrolPoints or {}
	self.CurrentPatrolIndex = 1
	self.HealthThreshold = 0.3
	self.LastAttackTime = 0
	self.Cooldown = 2
	return self
end

function AI:SetTarget(target)
	self.Target = target
end

function AI:SetState(newState)
	self.State = newState
end

function AI:DistanceToTarget()
	if self.Target and self.Target:IsDescendantOf(workspace) then
		return (self.NPC.Position - self.Target.Position).Magnitude
	end
	return math.huge
end

function AI:PlayAnimation(animName)
	local animator = self.NPC:FindFirstChildWhichIsA("Animator", true)
	if animator then
		local anim = self.NPC.Animations and self.NPC.Animations:FindFirstChild(animName)
		if anim then
			animator:LoadAnimation(anim):Play()
		end
	end
end

function AI:Update()
	if not self.NPC or not self.Target then return end

	local dist = self:DistanceToTarget()

	if self.State == "Idle" then
		if #self.PatrolPoints > 0 then
			self:SetState("Patrol")
		elseif dist < self.SightRange then
			self:SetState("Chase")
		end

	elseif self.State == "Patrol" then
		local targetPoint = self.PatrolPoints[self.CurrentPatrolIndex]
		if (self.NPC.Position - targetPoint.Position).Magnitude < 3 then
			self.CurrentPatrolIndex = self.CurrentPatrolIndex % #self.PatrolPoints + 1
		else
			local dir = (targetPoint.Position - self.NPC.Position).Unit
			self.NPC.Velocity = dir * 10
		end

		if dist < self.SightRange then
			self:SetState("Chase")
		end

	elseif self.State == "Chase" then
		local dir = (self.Target.Position - self.NPC.Position).Unit
		self.NPC.Velocity = dir * 16
		self:PlayAnimation("Run")

		if dist <= self.AttackRange then
			self:SetState("Attack")
		elseif dist > self.SightRange + 10 then
			self:SetState("Idle")
		end

	elseif self.State == "Attack" then
		if tick() - self.LastAttackTime >= self.Cooldown then
			self.LastAttackTime = tick()
			self:PlayAnimation("Attack")
		end

		if dist > self.AttackRange then
			self:SetState("Chase")
		end

	elseif self.State == "Retreat" then
		local dir = (self.NPC.Position - self.Target.Position).Unit
		self.NPC.Velocity = dir * 18
	end
end

function AI:BindToHeartbeat()
	RunService.Heartbeat:Connect(function()
		self:Update()
	end)
end

return AI
