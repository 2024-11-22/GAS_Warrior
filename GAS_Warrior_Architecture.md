# GAS_Warrior 项目代码结构分析

## 📁 文件结构

```
Source/
├── Warrior.Target.cs
├── Warrior/
│   ├── Warrior.Build.cs
│   ├── Warrior.h / Warrior.cpp
│   ├── Public/
│   │   ├── WarriorGameplayTags.h (输入/事件标签)
│   │   ├── WarriorFunctionLibrary.h (工具函数)
│   │   ├── WarriorDebugHelper.h (调试工具)
│   │   ├── WarriorGameInstance.h
│   │   ├── WarriorTypes/ (枚举/结构体)
│   │   ├── Interfaces/ (PawnCombatInterface, PawnUIInterface)
│   │   ├── Characters/ (角色类)
│   │   ├── Components/ (PawnExtension, Combat, UI, Input)
│   │   ├── Controllers/ (Hero, AI)
│   │   ├── GameModes/ (Base, Survival)
│   │   ├── DataAssets/ (StartUpData, Input)
│   │   ├── AbilitySystem/ (ASC, AttributeSet, Abilities, Tasks)
│   │   ├── AnimInstances/ (Base, Character, Hero)
│   │   ├── Items/ (Weapons, PickUps, Projectile)
│   │   ├── AI/ (BTTask, BTService)
│   │   ├── Widgets/ (WarriorWidgetBase)
│   │   └── SaveGame/
│   └── Private/ (对应的.cpp实现文件)
```

---

## 1. 类图 (Class Diagrams)

### 1.1 角色类继承关系

```
┌─────────────────────────────────────────────────────────────────────┐
│                           AActor                                     │
└─────────────────────────────────────────────────────────────────────┘
                                    △
                                    │
┌───────────────────────────────────────────────────────────────────┐
│                        ACharacter (UE5)                             │
└───────────────────────────────────────────────────────────────────┘
                                    △
                                    │
┌───────────────────────────────────────────────────────────────────┐
│                     AWarriorBaseCharacter                           │
├───────────────────────────────────────────────────────────────────┤
│ + WarriorAbilitySystemComponent : UWarriorAbilitySystemComponent* │
│ + AttributeSet : UWarriorAttributeSet*                             │
│ + MotionWarpingComponent : UMotionWarpingComponent*               │
│ + CharacterStartUpData : TSoftObjectPtr<UDataAsset_StartUpDataBase>│
├───────────────────────────────────────────────────────────────────┤
│ + SetupInputComponent()                                            │
│ + PossessedBy(AController*)                                        │
│ + GetAbilitySystemComponent() : UAbilitySystemComponent*           │
│ + GetPawnCombatInterface() : IPawnCombatInterface                  │
│ + GetPawnUIInterface() : IPawnUIInterface                          │
├───────────────────────────────────────────────────────────────────┤
│ «implements» IAbilitySystemInterface                                │
│ «implements» IPawnCombatInterface                                  │
│ «implements» IPawnUIInterface                                      │
└────────────────────────────┬──────────────────────────────────────┘
                             │
          ┌──────────────────┴──────────────────┐
          ▼                                     ▼
┌─────────────────────────┐     ┌─────────────────────────────────┐
│  AWarriorHeroCharacter   │     │    AWarriorEnemyCharacter      │
├─────────────────────────┤     ├─────────────────────────────────┤
│ + CameraBoom            │     │ + EnemyCombatComponent          │
│ + FollowCamera          │     │ + EnemyUIComponent              │
│ + HeroCombatComponent    │     │ + LeftHandCollisionBox         │
│ + HeroUIComponent       │     │ + RightHandCollisionBox        │
│ + InputConfigDataAsset   │     │ + EnemyHealthWidgetComponent   │
├─────────────────────────┤     └─────────────────────────────────┘
│ + Input_Move()          │
│ + Input_Look()          │
│ + Input_SwitchTarget()   │
│ + Input_PickUpStones()   │
│ + Input_AbilityPressed() │
│ + Input_AbilityReleased()│
└─────────────────────────┘
```

### 1.2 组件类继承关系

```
┌─────────────────────────────────────────────────────────────────────┐
│                     UActorComponent (UE5)                           │
└─────────────────────────────────────────────────────────────────────┘
                                    △
                                    │
                    ┌───────────────┴───────────────┐
                    ▼                               ▼
┌───────────────────────────────────┐   ┌───────────────────────────────────┐
│     UPawnExtensionComponentBase    │   │         UActorComponent           │
├───────────────────────────────────┤   └───────────────────────────────────┘
│ + GetOwningPawn<T>()              │
│ + GetOwningController<T>()        │
└───────────────┬───────────────────┘
                │
    ┌───────────┴───────────┬───────────────────┐
    ▼                       ▼                   ▼
┌──────────────┐   ┌───────────────┐   ┌────────────────────┐
│UPawnCombatComponent│  │UPawnUIComponent│   │ UWarriorInputComponent│
├──────────────┤   ├───────────────┤   └────────────────────┘
│+ CarriedWeaponMap   │  │+ OnCurrentHealth│
│+ CurrentEquipWeapon │  │   Changed       │
│+ OverlappedActors   │  └───────┬─────────┘
├──────────────┤            ┌────┴────┐
│+ RegisterSpawnedWeapon()│        │
│+ GetCharacterCarriedWeaponByTag()│   ┌────────────────┐
│+ GetCharacterCurrentEquippedWeapon│   │ UHeroUIComponent│
│+ ToggleWeaponCollision()        │   ├────────────────┤
│+ OnHitTargetActor()            │   │+ OnRageChanged   │
│+ OnWeaponPulledFromTargetActor()│   │+ OnWeaponChanged │
└───────┬──────────┘            │+ OnAbilityUpdated│
        │                       │+ OnCooldownBegin  │
┌───────┴───────────────┐      │+ OnStoneInteracted│
▼                       ▼      └────────┬─────────┘
┌─────────────────┐  ┌─────────────────────┐
│UHeroCombatComponent│ │UEnemyCombatComponent│
├─────────────────┤  ├─────────────────────┤
│+ GetHeroCarriedWeapon()│ │ (继承父类方法)  │
│+ GetHeroCurrentEquippedWeapon()│           │
│+ GetHeroCurrentEquippedWeaponDamageAtLevel()│            │
└─────────────────┘  └─────────────────────┘
```

### 1.3 能力系统类继承关系

```
┌─────────────────────────────────────────────────────────────────────┐
│                   UGameplayAbility (GameplayAbilities)              │
└────────────────────────────────────┬────────────────────────────────┘
                                    △
                                    │
                    ┌───────────────┴───────────────┐
                    ▼                               ▼
┌───────────────────────────────────┐   ┌───────────────────────────────────┐
│    UWarriorGameplayAbility        │   │    UWarriorEnemyGameplayAbility   │
├───────────────────────────────────┤   ├───────────────────────────────────┤
│ + NativeApplyEffectSpecHandleToTarget()│   │ (继承父类能力逻辑)              │
│ + ApplyGameplayEffectSpecHandleToHitResults()│                                   │
│ + GetPawnCombatComponentFromActorInfo()│                                  │
│ + GetWarriorAbilitySystemComponentFromActorInfo()│                          │
│ + ActivationPolicy : EWarriorAbilityActivationPolicy│                          │
├───────────────────────────────────┤   └───────────────────────────────────┘
│ «enum» EWarriorAbilityActivationPolicy│
│   - OnTriggered                     │
│   - OnGiven                          │
└───────────────┬─────────────────────┘
                │
                ▼
┌───────────────────────────────────┐
│  UWarriorHeroGameplayAbility      │
├───────────────────────────────────┤
│ + GetHeroCharacterFromActorInfo() │
│ + GetHeroControllerFromActorInfo()│
│ + GetHeroCombatComponentFromActorInfo()│
│ + GetHeroUIComponentFromActorInfo()│
│ + MakeHeroDamageEffectSpecHandle()│
└───────────────┬───────────────────┘
                │
    ┌───────────┴────────────────┐
    ▼                            ▼
┌────────────────────────┐ ┌─────────────────────────────┐
│UHeroGameplayAbility_   │ │UHeroGameplayAbility_        │
│TargetLock               │ │PickUpStones                  │
├────────────────────────┤ ├─────────────────────────────┤
│+ BoxTraceDistance      │ │ (石头拾取逻辑)               │
│+ TargetLockRotation... │ │                             │
│+ TargetLockMaxWalkSpeed│ │                             │
│+ TargetLockMappingContext│ │                             │
│+ TryLockOnTarget()     │ │                             │
│+ SwitchTarget()        │ │                             │
│+ OnTargetLockTick()    │ │                             │
│+ UnlockTarget()        │ │                             │
└────────────────────────┘ └─────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                 UAbilitySystemComponent (UE5 GAS)                   │
└────────────────────────────────────┬────────────────────────────────┘
                                    △
                                    │
                    ┌───────────────┴────────────────┐
                    ▼                                ▼
┌───────────────────────────────────┐   ┌───────────────────────────────────┐
│  UWarriorAbilitySystemComponent   │   │      UAttributeSet                │
├───────────────────────────────────┤   ├───────────────────────────────────┤
│ + OnAbilityInputPressed()         │   │ + CurrentHealth / MaxHealth       │
│ + OnAbilityInputReleased()        │   │ + CurrentRage / MaxRage           │
│ + GrantHeroWeaponAbilities()      │   │ + AttackPower / DefensePower      │
│ + RemoveGrantHeroWeaponAbilities()│   │ + DamageTaken                    │
│ + TryActivateAbilityByTag()       │   ├───────────────────────────────────┤
└───────────────────────────────────┘   │ + PostGameplayEffectExecute()     │
                                        └───────────────────────────────────┘
```

### 1.4 武器与物品类关系

```
┌─────────────────────────────────────────────────────────────────────┐
│                           AActor                                     │
└─────────────────────────────────────────────────────────────────────┘
                                    △
                                    │
                    ┌───────────────┴───────────────┐
                    ▼                               ▼
┌───────────────────────────────────┐   ┌───────────────────────────────────┐
│         AWarriorWeaponBase        │   │      AWarriorPickUpBase           │
├───────────────────────────────────┤   ├───────────────────────────────────┤
│ + WeaponMesh : UStaticMeshComponent│  │ + PickUpCollisionSphere          │
│ + WeaponCollisionBox : UBoxComponent│ │                                   │
├───────────────────────────────────┤   └───────────────┬───────────────────┘
│ + OnWeaponHitTarget : FOnWeaponHit│                   │
│ + OnWeaponPulledFromTarget        │                   ▼
└───────────────────────────────────┘   ┌───────────────────────────────────┐
                                        │        AWarriorStoneBase          │
                                        ├───────────────────────────────────┤
┌───────────────────────────────────┐   │ + StoneGameplayEffectClass       │
│       AWarriorHeroWeapon          │   └───────────────────────────────────┘
├───────────────────────────────────┤
│ + HeroWeaponData                  │
└───────────────────────────────────┘
```

### 1.5 接口定义

```
┌─────────────────────────────────────────────────────────────────────┐
│                        IPawnCombatInterface                          │
├─────────────────────────────────────────────────────────────────────┤
│ + GetPawnCombatComponent() : UPawnCombatComponent*                  │
└─────────────────────────────────────────────────────────────────────┘
                                    △
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         IPawnUIInterface                             │
├─────────────────────────────────────────────────────────────────────┤
│ + GetPawnUIComponent() : UPawnUIComponent*                          │
└─────────────────────────────────────────────────────────────────────┘

实现类:
  • AWarriorBaseCharacter
  • AWarriorHeroCharacter
  • AWarriorEnemyCharacter
```

---

## 2. 模块图 (Module Diagram)

### 2.1 整体架构模块图

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                          WARRIOR 核心模块架构                                  │
└──────────────────────────────────────────────────────────────────────────────┘

    ┌─────────────────────────────────────────────────────────────────────┐
    │                      Input System (输入系统)                        │
    │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐   │
    │  │GameplayTags     │  │InputConfig      │  │InputComponent   │   │
    │  │(输入事件定义)    │  │(输入映射配置)    │  │(输入绑定处理)    │   │
    │  └─────────────────┘  └─────────────────┘  └─────────────────┘   │
    └─────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
    ┌─────────────────────────────────────────────────────────────────────┐
    │                     Character System (角色系统)                     │
    │  ┌───────────────┐    ┌───────────────┐    ┌───────────────┐       │
    │  │ BaseCharacter │───▶│ HeroCharacter │    │EnemyCharacter │      │
    │  └───────┬───────┘    └───────────────┘    └───────┬───────┘       │
    │          │                                               │          │
    │          ▼                                               ▼          │
    │  ┌─────────────────────────────────────────────────────────────┐    │
    │  │                    Component Layer                          │    │
    │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │    │
    │  │  │ CombatComp  │  │   UIComp    │  │ InputComp   │         │    │
    │  │  │ (战斗组件)   │  │  (UI组件)    │  │ (输入组件)   │         │    │
    │  │  └─────────────┘  └─────────────┘  └─────────────┘         │    │
    │  └─────────────────────────────────────────────────────────────┘    │
    └─────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
    ┌─────────────────────────────────────────────────────────────────────┐
    │               Gameplay Ability System (GAS能力系统)                 │
    │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌──────────┐ │
    │  │ ASC         │  │ AttributeSet│  │ Abilities   │  │ GE/Exec  │ │
    │  │(能力组件)    │  │ (属性集)     │  │ (技能)       │  │ (效果计算)│ │
    │  └─────────────┘  └─────────────┘  └─────────────┘  └──────────┘ │
    └─────────────────────────────────────────────────────────────────────┘
          │               │               │
          ▼               ▼               ▼
    ┌─────────────────────────────────────────────────────────────────────┐
    │                     Game Mode & AI (游戏模式与AI)                   │
    │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                 │
    │  │BaseGameMode │  │SurvivalMode │  │   AI        │                 │
    │  │ (基础模式)   │  │  (生存波次)  │  │ (行为树/感知)│                 │
    │  └─────────────┘  └─────────────┘  └─────────────┘                 │
    └─────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│                           Support Modules (支撑模块)                        │
├──────────────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │  Animation   │  │    Items     │  │   Widgets    │  │  SaveGame    │   │
│  │ (动画实例)    │  │ (武器/拾取)   │  │  (UI组件)    │  │  (存档系统)   │   │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘   │
└──────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 模块依赖关系图

```
                          ┌─────────────────────────────┐
                          │  WarriorFunctionLibrary     │
                          │      (全局工具函数)           │
                          └──────────────┬──────────────┘
                                         │
     ┌───────────────────────────────────┼───────────────────────────────────┐
     │                                   │                                   │
     ▼                                   ▼                                   ▼
┌────────────┐                   ┌────────────┐                    ┌────────────┐
│   Input    │                   │   Combat   │                    │     UI     │
│   Module   │                   │   Module   │                    │   Module   │
├────────────┤                   ├────────────┤                    ├────────────┤
│• InputConfig│                   │• CombatComp│                    │• PawnUIComp│
│• GameplayTags│                  │• HeroCombat │                   │• HeroUIComp│
│• InputComp │                   │• EnemyCombat│                   │• EnemyUIComp│
└─────┬──────┘                   └──────┬─────┘                    └──────┬─────┘
      │                                 │                                 │
      └────────────────────┬────────────┴─────────────────────────────────┘
                           │
                           ▼
            ┌──────────────────────────────┐
            │      Ability System (GAS)     │
            ├──────────────────────────────┤
            │• WarriorAbilitySystemComp    │
            │• WarriorAttributeSet        │
            │• GameplayAbilities           │
            │• GameplayEffects            │
            │• GEExecCalc_DamageTaken     │
            └──────────────┬───────────────┘
                           │
     ┌─────────────────────┼─────────────────────┐
     │                     │                     │
     ▼                     ▼                     ▼
┌────────────┐     ┌────────────┐        ┌────────────┐
│ Character  │     │     AI    │        │  GameMode  │
│   Module   │     │   Module   │        │   Module   │
├────────────┤     ├────────────┤        ├────────────┤
│• BaseChar  │     │• AIController│      │• BaseGameMode│
│• HeroChar  │     │• BTTasks   │        │• SurvivalGameMode│
│• EnemyChar │     │• BTServices│        │              │
└────────────┘     └────────────┘        └────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                          Interface Layer (接口层)                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│    IPawnCombatInterface ◀───────────── 被所有Pawn实现                      │
│           │                                                                 │
│           └────────▶ UPawnCombatComponent::GetPawnCombatComponent()       │
│                                                                             │
│    IPawnUIInterface ◀────────────── 被所有Pawn实现                         │
│           │                                                                 │
│           └────────▶ UPawnUIComponent::GetPawnUIComponent()                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. 时序图 (Sequence Diagrams)

### 3.1 武器攻击流程

```
┌────────┐    ┌───────────────┐   ┌─────────────┐   ┌────────────────┐   ┌────────────┐
│ Player │    │HeroController │   │InputComponent│   │AbilitySystemComp│   │  Ability   │
└───┬────┘    └───────┬───────┘   └──────┬──────┘   └───────┬────────┘   └─────┬──────┘
    │                 │                   │                  │                 │
    │ Input: Light Attack                 │                  │                 │
    │────────────────▶│                   │                  │                 │
    │                 │                   │                  │                 │
    │                 │                   │                  │                 │
    │                 │ Input_AbilityPressed(tag)             │                 │
    │                 │──────────────────▶│                  │                 │
    │                 │                   │                  │                 │
    │                 │                   │ OnAbilityInputPressed(tag)         │
    │                 │                   │────────────────▶│                 │
    │                 │                   │                  │                 │
    │                 │                   │                  │ TryActivateAbility
    │                 │                   │                  │────────────────▶│
    │                 │                   │                  │                 │
    │                 │                   │                  │◀────────────────│
    │                 │                   │                  │  Ability激活     │
    │                 │                   │                  │                 │
    │◀────────────────────────────────────────────────────────────────────────│
    │         播放攻击动画 (Montage)                  │                 │
    │                 │                   │                  │                 │
    │                 │                   │                  │                 │
    │                 │                   │ 启用武器碰撞      │                 │
    │                 │                   │◀─────────────────│                 │
    │                 │                   │                  │                 │
    │                 │                   │                  │                 │
    │   [动画播放中 - 武器碰撞盒激活]         │                  │                 │
    │                 │                   │                  │                 │
    │                 │                   │                  │                 │
    │                 │                   │              ┌───┴─────┐         │
    │                 │                   │              │Weapon   │         │
    │                 │                   │              │Collision│         │
    │                 │                   │              │  Box    │         │
    │                 │                   │              └────┬────┘         │
    │                 │                   │                  │                 │
    │                 │                   │                  │ 命中检测       │
    │                 │                   │                  │◀──────────────│
    │                 │                   │                  │                 │
    │                 │                   │ OnHitTargetActor │                 │
    │                 │                   │◀──────────────────────────────────│
    │                 │                   │                  │                 │
    │                 │                   │ 发送GameplayEvent(对谁,做什么)     │
    │                 │                   │────────────────▶│                 │
    │                 │                   │                  │                 │
    │                 │                   │                  │ 应用伤害Effect  │
    │                 │                   │                  │────────▶┌───────┐
    │                 │                   │                  │         │AttrSet│
    │                 │                   │                  │         └───┬───┘
    │                 │                   │                  │             │
    │                 │                   │                  │ 属性更新: -Health
    │                 │                   │                  │             │
    │                 │                   │                  │             │
    │                 │                   │                  │ OnHealthChanged
    │                 │                   │                  │◀──────────────│
    │                 │                   │                  │                 │
    │                 │                   │                  │ 广播UI更新    │
    │                 │                   │                  │────────────────▶│
    │                 │                   │                  │                 │
    │                 │                   │                  │                 │
    │                 │                   │                  │                 │
    └─────────────────┴───────────────────┴──────────────────┴─────────────────┘
                                  攻击流程结束
```

### 3.2 目标锁定流程

```
┌────────┐    ┌───────────────┐   ┌─────────────┐   ┌────────────────┐   ┌────────────┐
│ Player │    │HeroCharacter │   │TargetLock   │   │   ASC          │   │  Widget    │
│        │    │               │   │ Ability     │   │                │   │ Manager    │
└───┬────┘    └───────┬───────┘   └──────┬──────┘   └───────┬────────┘   └─────┬──────┘
    │                 │                   │                  │                 │
    │ 按下锁定键       │                   │                  │                 │
    │────────────────▶│                   │                  │                 │
    │                 │                   │                  │                 │
    │                 │ TryLockOnTarget() │                  │                 │
    │                 │──────────────────▶│                  │                 │
    │                 │                   │                  │                 │
    │                 │                   │ BoxTraceMulti    │                 │
    │                 │                   │─────────────────│                 │
    │                 │                   │◀─────────────────│                 │
    │                 │                   │ 命中目标列表      │                 │
    │                 │                   │                  │                 │
    │                 │                   │ FindNearestActor │                 │
    │                 │                   │ (获取最近目标)    │                 │
    │                 │                   │                  │                 │
    │                 │                   │ DrawTargetLockWidget              │
    │                 │                   │─────────────────────────────────▶│
    │                 │                   │                  │                 │
    │                 │◀──────────────────│ 锁定成功         │                 │
    │                 │                   │                  │                 │
    │                 │ 锁定状态: bIsTargetLocked=true       │                 │
    │                 │                   │                  │                 │
    │                 │                   │                  │                 │
    │◀────────────────────────────────────────────────────────────────────────│
    │       进入Tick循环                        │                  │                 │
    │                 │                   │                  │                 │
    │                 │                   │ OnTargetLockTick (每帧)             │
    │                 │                   │◀─────────────────────────────────│
    │                 │                   │                  │                 │
    │                 │                   │ 更新Widget位置   │                 │
    │                 │                   │────────────────▶│                 │
    │                 │                   │                  │                 │
    │                 │                   │ 旋转角色面向目标  │                 │
    │                 │                   │◀─────────────────│                 │
    │                 │                   │                  │                 │
    │                 │                   │ 限制移动速度     │                 │
    │                 │                   │                  │                 │
    │   [玩家正常游戏 - 视角跟随锁定目标]      │                  │                 │
    │                 │                   │                  │                 │
    │────────────────▶│                   │                  │                 │
    │ 按下切换键       │                   │                  │                 │
    │                 │                   │                  │                 │
    │                 │ SwitchTarget(direction)              │                 │
    │                 │──────────────────▶│                  │                 │
    │                 │                   │                  │                 │
    │                 │                   │ GetAvailableTargets              │
    │                 │                   │ (获取同侧目标)    │                 │
    │                 │                   │                  │                 │
    │                 │                   │ SetCurrentLockedActor            │
    │                 │                   │ (切换锁定目标)    │                 │
    │                 │                   │                  │                 │
    │◀────────────────────────────────────────────────────────────────────────│
    │                 │                   │                  │                 │
    │   [继续锁定流程] │                   │                  │                 │
    │                 │                   │                  │                 │
    │────────────────▶│                   │                  │                 │
    │ 再次按下锁定键   │                   │                  │                 │
    │                 │                   │                  │                 │
    │                 │ UnlockTarget()   │                  │                 │
    │                 │──────────────────▶│                  │                 │
    │                 │                   │                  │                 │
    │                 │                   │ 销毁Widget      │                 │
    │                 │                   │────────────────▶│                 │
    │                 │                   │                  │                 │
    │                 │◀──────────────────│ 解锁成功         │                 │
    │                 │                   │                  │                 │
    │                 │ 锁定状态: bIsTargetLocked=false      │                 │
    │                 │                   │                  │                 │
    └─────────────────┴───────────────────┴──────────────────┴─────────────────┘
```

### 3.3 敌人生成波次流程

```
┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│SurvivalGame │  │  DataTable  │  │    Spawner   │  │   Enemy     │  │ GameInstance │
│    Mode     │  │             │  │              │  │             │  │             │
└──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘
       │                │                │                │                │
       │ [状态: WaitSpawnNewWave]         │                │                │
       │                │                │                │                │
       │ Tick()         │                │                │                │
       │──────────────▶ │                │                │                │
       │                │                │                │                │
       │ 读取波次配置    │                │                │                │
       │───────────────▶│                │                │                │
       │◀───────────────│                │                │                │
       │ WaveData        │                │                │                │
       │                │                │                │                │
       │ [状态: SpawningNeWWave]         │                │                │
       │                │                │                │                │
       │ PreLoadNextWaveEnemies         │                │                │
       │─────────────────────────────────────────────────────────────────▶│
       │                │                │                │                │
       │                │                │◀──────────────────────────────────│
       │                │                │   敌人Class加载完成              │
       │                │                │                │                │
       │ TrySpawnWaveEnemies            │                │                │
       │──────────────▶│                │                │                │
       │                │                │                │                │
       │                │                │ SpawnEnemy(位置)│                │
       │                │                │──────────────▶│                │
       │                │                │                │                │
       │                │                │◀──────────────│                │
       │                │                │  Enemy Spawned│                │
       │                │                │                │                │
       │ RegisterSpawnedEnemy           │                │                │
       │                │                │                │                │
       │                │                │                │                │
       │ [状态: InProgress]              │                │                │
       │                │                │                │                │
       │                │                │                │  [战斗中...]  │
       │                │                │                │                │
       │                │                │                │                │
       │ OnEnemyDestroyed◀────────────────────────────────────────────────│
       │                │                │                │                │
       │ DecrementEnemyCount            │                │                │
       │                │                │                │                │
       │ EnemyCount == 0?               │                │                │
       │                │                │                │                │
       │ [YES]          │                │                │                │
       │                │                │                │                │
       │ [状态: WaveCompleted]          │                │                │
       │                │                │                │                │
       │ 下一波可用?     │                │                │                │
       │                │                │                │                │
       │ [YES]          │                │                │                │
       │                │                │                │                │
       │ [状态: WaitSpawnNewWave]        │                │                │
       │─────────────── │                │                │                │
       │                │                │                │                │
       │ [NO - 全部完成] │                │                │                │
       │                │                │                │                │
       │ [状态: AllWavesDone]           │                │                │
       │                │                │                │                │
       │ 触发结算流程    │                │                │                │
       │                │                │                │                │
       └────────────────┴────────────────┴────────────────┴────────────────┘
```

### 3.4 能力系统输入处理流程

```
┌────────┐    ┌────────┐    ┌────────┐    ┌───────────────┐    ┌────────────┐
│ Player │    │ Input  │    │ Hero   │    │ AbilitySystem │    │  Ability   │
│        │    │Action  │    │Character│   │  Component    │    │            │
└───┬────┘    └───┬────┘    └───┬────┘    └───────┬───────┘    └─────┬──────┘
    │              │            │                  │                 │
    │ InputEvent   │            │                  │                 │
    │─────────────▶│            │                  │                 │
    │              │            │                  │                 │
    │ Triggered    │            │                  │                 │
    │─────────────▶│            │                  │                 │
    │              │            │                  │                 │
    │              │ Input_AbilityPressed(tag)     │                 │
    │              │───────────▶│                  │                 │
    │              │            │                  │                 │
    │              │            │ OnAbilityInputPressed(tag)         │
    │              │            │──────────────────▶│                 │
    │              │            │                  │                 │
    │              │            │                  │ GetAbilityByTag │
    │              │            │                  │────────────────▶│
    │              │            │                  │◀────────────────│
    │              │            │                  │                 │
    │              │            │                  │ HandleAbilityInput│
    │              │            │                  │                 │
    │              │            │                  │ [激活状态=OnTriggered]│
    │              │            │                  │                 │
    │              │            │                  │ CanActivateAbility?│
    │              │            │                  │◀────────────────│
    │              │            │                  │                 │
    │              │            │                  │ ActivateAbility│
    │              │            │                  │────────────────▶│
    │              │            │                  │                 │
    │              │            │                  │ 能力执行...     │
    │              │            │                  │◀────────────────│
    │              │            │                  │                 │
    │◀────────────────────────────────────────────────────────────────│
    │    Input Released                          │                 │
    │─────────────▶│            │                  │                 │
    │              │            │                  │                 │
    │              │ Input_AbilityReleased(tag)   │                 │
    │              │───────────▶│                  │                 │
    │              │            │                  │                 │
    │              │            │ OnAbilityInputReleased(tag)        │
    │              │            │──────────────────▶│                 │
    │              │            │                  │                 │
    │              │            │                  │ CancelAbility  │
    │              │            │                  │ (如需要)        │
    │              │            │                  │                 │
    └──────────────┴────────────┴──────────────────┴─────────────────┘
```

---

## 4. 数据流图

### 4.1 伤害处理数据流

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          DAMAGE DATA FLOW                                   │
└─────────────────────────────────────────────────────────────────────────────┘

    ┌────────────┐                    ┌─────────────────┐
    │   Ability   │                    │  Input System   │
    └──────┬──────┘                    └─────────────────┘
           │                                    ▲
           ▼                                    │
    ┌────────────┐                    ┌─────────────────┐
    │  Weapon    │                    │  DataAsset      │
    │  Collision │                    │  InputConfig    │
    └──────┬──────┘                    └─────────────────┘
           │                                    │
           │ HitResult                          │
           ▼                                    │
    ┌─────────────────────────────────────────────┐
    │              HeroCombatComponent            │
    │  ┌─────────────────────────────────────┐   │
    │  │ 1. OnHitTargetActor(HitResult)      │   │
    │  │ 2. 获取武器和基础伤害值              │   │
    │  │ 3. 构建DamageEffectParams           │   │
    │  └─────────────────────────────────────┘   │
    └─────────────────────┬─────────────────────┘
                          │
                          │ DamageEffectParams
                          ▼
    ┌─────────────────────────────────────────────┐
    │        WarriorHeroGameplayAbility           │
    │  ┌─────────────────────────────────────┐   │
    │  │ MakeHeroDamageEffectSpecHandle()   │   │
    │  │ • SourceAbilitySystemComponent      │   │
    │  │ • SourceActor (武器持有者)           │   │
    │  │ • TargetActor (被击中者)             │   │
    │  │ • BaseDamage (来自武器)             │   │
    │  │ • DamageType (物理/魔法等)           │   │
    │  └─────────────────────────────────────┘   │
    └─────────────────────┬─────────────────────┘
                          │
                          │ GameplayEffectSpecHandle
                          ▼
    ┌─────────────────────────────────────────────┐
    │        GEExecCalc_DamageTaken              │
    │  ┌─────────────────────────────────────┐   │
    │  │ Execute:                            │   │
    │  │ FinalDamage = BaseDamage            │   │
    │  │           × (1 + AttackPower/100)  │   │
    │  │           × (1 - DefensePower/200) │   │
    │  └─────────────────────────────────────┘   │
    └─────────────────────┬─────────────────────┘
                          │
                          │ Calculated Damage
                          ▼
    ┌─────────────────────────────────────────────┐
    │           WarriorAttributeSet               │
    │  ┌─────────────────────────────────────┐   │
    │  │ PostGameplayEffectExecute()         │   │
    │  │ CurrentHealth -= FinalDamage        │   │
    │  │ CurrentHealth = FMath::Max(0)       │   │
    │  └─────────────────────────────────────┘   │
    │                                              │
    │  广播事件: OnCurrentHealthChanged           │
    │                                              │
    └─────────────────────┬─────────────────────┘
                          │
                          │ HealthChangedEvent
                          ▼
    ┌─────────────────────────────────────────────┐
    │              HeroUIComponent                │
    │  ┌─────────────────────────────────────┐   │
    │  │ OnCurrentHealthChanged.Broadcast()  │   │
    │  │ • 更新血条UI                        │   │
    │  │ • 检查死亡状态                      │   │
    │  └─────────────────────────────────────┘   │
    └─────────────────────────────────────────────┘
```

---

## 5. 状态机图

### 5.1 生存模式状态机

```
┌─────────────────┐
│   StartGame     │
└────────┬────────┘
         │
         ▼
┌─────────────────────────┐
│   WaitSpawnNewWave       │
│  (等待生成新波次)         │
└────────┬────────────────┘
         │
    时间到?/手动触发
         │
         ▼
┌─────────────────────────┐
│   SpawningNeWWave       │
│  (正在生成敌人群)        │
└────────┬────────────────┘
         │
    敌人生成完成
         │
         ▼
┌─────────────────────────┐
│     InProgress          │◀──────────────────┐
│   (波次进行中)           │                    │
└────────┬────────────────┘                    │
         │                                       │
    所有敌人死亡?                                  │
         │                                       │
         ▼                                       │
┌─────────────────────────┐                      │
│    WaveCompleted        │                      │
│   (波次完成)             │                      │
└────────┬────────────────┘                      │
         │                                       │
    还有下一波?                                    │
    │                                             │
    │YES                                          │
    ▼                                             │
┌─────────────────────────┐                      │
│   WaitSpawnNewWave       │──────────────────────┘
│   (下一波)               │
└─────────────────────────┘

    NO
    │
    ▼
┌─────────────────────────┐
│    AllWavesDone         │
│   (全部波次完成)         │
└─────────────────────────┘

┌─────────────────────────┐
│     PlayerDied          │
│   (玩家死亡)             │
└─────────────────────────┘
```

### 5.2 目标锁定状态机

```
┌─────────────────────────┐
│      Idle               │
│   (未锁定状态)           │
└────────┬────────────────┘
         │
    按下锁定键
         │
         ▼
┌─────────────────────────┐
│   SearchingTarget       │
│  (搜索目标中)            │
└────────┬────────────────┘
         │
    找到目标?
    │           │
    │YES        │NO
    ▼           │
┌──────────────┴──────────┐
│      Locked            │
│   (已锁定状态)           │
└────────┬───────────────┘
         │
    ┌─────┴─────┐
    │           │
    ▼           ▼
 按切换键    再次按下锁定键
    │           │
    ▼           ▼
┌────────┐  ┌─────────────────┐
│Switching│  │    Unlocking    │
│Target   │  │    (解锁)       │
└────┬────┘  └────────┬────────┘
     │                 │
     └───────┬─────────┘
             │
             ▼
        ┌──────────────┐
        │   Locked     │
        │  (重新锁定)   │
        └──────────────┘
```

---

## 6. 总结

### 6.1 核心设计模式

| 模式 | 应用场景 |
|------|----------|
| **Component Pattern** | 战斗、UI、输入组件解耦 |
| **Interface Pattern** | IPawnCombatInterface, IPawnUIInterface |
| **Observer Pattern** | Delegate/Broadcast事件系统 |
| **State Pattern** | 生存模式波次状态机 |
| **Template Method** | PawnExtensionComponentBase::GetOwningPawn<T>() |
| **Data-Driven** | DataTable配置波次、DataAsset配置输入 |

### 6.2 技术栈

| 领域 | 技术 |
|------|------|
| 游戏引擎 | Unreal Engine 5 |
| 能力系统 | GameplayAbilitySystem (GAS) |
| 输入系统 | Enhanced Input System |
| AI系统 | BehaviorTree, AIPerception |
| 网络同步 | Replicated, NetCull |
| 动画系统 | AnimInstance, LinkedAnimLayer |
