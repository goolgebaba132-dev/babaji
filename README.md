package com.aadhyanshgamerz.capes;

import net.fabricmc.api.ClientModInitializer;
import net.fabricmc.fabric.api.client.keybinding.v1.KeyBindingHelper;
import net.fabricmc.fabric.api.client.rendering.v1.PlayerRenderCallback;
import net.minecraft.client.MinecraftClient;
import net.minecraft.client.option.KeyBinding;
import net.minecraft.client.network.AbstractClientPlayerEntity;
import net.minecraft.util.Identifier;
import org.lwjgl.glfw.GLFW;

import java.util.LinkedHashMap;
import java.util.Map;

/**
 * Aadhyanshgamerz Capes Mod
 * Minecraft 1.21.1 | Fabric 0.18.0
 */
public class AadhyanshgamerzCapesClient implements ClientModInitializer {

    /* ---------------- CAPE DATA ---------------- */

    private static final Map<String, Identifier> STATIC_CAPES = new LinkedHashMap<>();
    private static final Map<String, AnimatedCape> ANIMATED_CAPES = new LinkedHashMap<>();

    private static String selectedCape = "minecon_animated";
    private static KeyBinding switchKey;

    /* ---------------- INIT ---------------- */

    @Override
    public void onInitializeClient() {
        registerCapes();
        registerKey();
        registerRenderer();
    }

    /* ---------------- CAPES ---------------- */

    private static void registerCapes() {
        STATIC_CAPES.put(
                "minecon_2011",
                new Identifier("aadhyanshgamerzcapes", "textures/capes/minecon_2011.png")
        );

        ANIMATED_CAPES.put(
                "minecon_animated",
                new AnimatedCape(
                        "textures/capes/animated/minecon",
                        10, // frames
                        2   // speed (ticks)
                )
        );
    }

    /* ---------------- KEYBIND ---------------- */

    private static void registerKey() {
        switchKey = KeyBindingHelper.registerKeyBinding(new KeyBinding(
                "key.aadhyanshgamerzcapes.switch",
                GLFW.GLFW_KEY_C,
                "category.aadhyanshgamerzcapes"
        ));
    }

    /* ---------------- RENDER ---------------- */

    private static void registerRenderer() {
        PlayerRenderCallback.EVENT.register((player, matrices, vertexConsumers, light) -> {
            if (!(player instanceof AbstractClientPlayerEntity p)) return;

            Identifier cape = getCurrentCape();
            if (cape != null) {
                p.setCapeTexture(cape);
            }

            // Switch cape with key
            if (switchKey.wasPressed()) {
                toggleCape();
            }
        });
    }

    /* ---------------- LOGIC ---------------- */

    private static Identifier getCurrentCape() {
        if (ANIMATED_CAPES.containsKey(selectedCape)) {
            return ANIMATED_CAPES.get(selectedCape).getFrame();
        }
        return STATIC_CAPES.get(selectedCape);
    }

    private static void toggleCape() {
        selectedCape = selectedCape.equals("minecon_animated")
                ? "minecon_2011"
                : "minecon_animated";
    }

    /* ---------------- ANIMATED CAPE CLASS ---------------- */

    private static class AnimatedCape {
        private final String basePath;
        private final int frames;
        private final int speed;

        public AnimatedCape(String basePath, int frames, int speed) {
            this.basePath = basePath;
            this.frames = frames;
            this.speed = speed;
        }

        public Identifier getFrame() {
            if (MinecraftClient.getInstance().world == null) return null;
            long time = MinecraftClient.getInstance().world.getTime();
            int frame = (int) ((time / speed) % frames);
            return new Identifier(
                    "aadhyanshgamerzcapes",
                    basePath + "/frame_" + frame + ".png"
            );
        }
    }
}
