# CloudFPS Mod - Vollständige Projektdateien

Hier findest du alle benötigten Dateien für deine Fabric-Mod zur Performance-Optimierung und Fast-Use-Mechanik (Fast Crystal, Fast Anchor, Fast Pearl).

---

## 1. Mod-Konfiguration (`src/main/resources/fabric.mod.json`)

```json
{
  "schemaVersion": 1,
  "id": "cloudfps",
  "version": "1.0.0",
  "name": "CloudFPS",
  "description": "Eine Mod zur Performance-Optimierung und Fast-Use-Mechanik.",
  "authors": [
    "CloudFPS Team"
  ],
  "contact": {},
  "license": "MIT",
  "icon": "assets/cloudfps/icon.png",
  "environment": "client",
  "entrypoints": {
    "fabric-client": [
      "de.cloudfps.CloudFpsClient"
    ]
  },
  "mixins": [
    "cloudfps.mixins.json"
  ],
  "depends": {
    "fabricloader": ">=0.15.0",
    "minecraft": "~1.21.1",
    "java": ">=21"
  }
}
```

---

## 2. Mixin-Konfiguration (`src/main/resources/cloudfps.mixins.json`)

```json
{
  "required": true,
  "package": "de.cloudfps.mixin",
  "compatibilityLevel": "JAVA_21",
  "client": [
    "ParticleManagerMixin",
    "GameRendererMixin",
    "BlockStateMixin",
    "MinecraftClientMixin"
  ],
  "injectors": {
    "defaultRequire": 1
  }
}
```

---

## 3. Client Initializer (`src/main/java/de/cloudfps/CloudFpsClient.java`)

```java
package de.cloudfps;

import net.fabricmc.api.ClientModInitializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class CloudFpsClient implements ClientModInitializer {
    public static final String MOD_ID = "cloudfps";
    public static final Logger LOGGER = LoggerFactory.getLogger(MOD_ID);

    @Override
    public void onInitializeClient() {
        LOGGER.info("CloudFPS wurde erfolgreich gestartet!");
    }
}
```

---

## 4. Fast Crystal, Anchor & Pearl Mixin (`src/main/java/de/cloudfps/mixin/MinecraftClientMixin.java`)

```java
package de.cloudfps.mixin;

import net.minecraft.client.MinecraftClient;
import net.minecraft.item.EnderPearlItem;
import net.minecraft.item.EndCrystalItem;
import net.minecraft.item.RespawnAnchorItem;
import net.minecraft.item.ItemStack;
import org.spongepowered.asm.mixin.Mixin;
import org.spongepowered.asm.mixin.Shadow;
import org.spongepowered.asm.mixin.injection.At;
import org.spongepowered.asm.mixin.injection.Inject;
import org.spongepowered.asm.mixin.injection.callback.CallbackInfo;

@Mixin(MinecraftClient.class)
public class MinecraftClientMixin {

    @Shadow
    private int itemUseCooldown;

    @Inject(method = "doItemUse", at = @At("HEAD"))
    private void onDoItemUse(CallbackInfo ci) {
        MinecraftClient client = (MinecraftClient) (Object) this;
        if (client.player == null) return;

        ItemStack mainHand = client.player.getMainHandStack();
        ItemStack offHand = client.player.getOffHandStack();

        // Wenn Crystal, Anchor oder Pearl gehalten werden, Cooldown auf 0 setzen
        if (isFastItem(mainHand) || isFastItem(offHand)) {
            this.itemUseCooldown = 0;
        }
    }

    private boolean isFastItem(ItemStack stack) {
        if (stack.isEmpty()) return false;
        return stack.getItem() instanceof EndCrystalItem ||
               stack.getItem() instanceof RespawnAnchorItem ||
               stack.getItem() instanceof EnderPearlItem;
    }
}
```