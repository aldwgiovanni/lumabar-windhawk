# Options : mods Windhawk pour se rapprocher de lumabar et de Seelen UI

Inventaire vérifié le 2026-11-04 contre le catalogue Windhawk (chaque id existe sur windhawk.net/mods/<id>) et la source de chaque mod. Les réglages sont au format **Textual mode** de l'onglet Settings ; les clés omises gardent leur valeur par défaut.

Ne pas combiner avec « Taskbar height and icon size » : le thème fixe déjà la hauteur de la barre.

## Écran à 125 %

Toutes les valeurs des thèmes sont en DIP (unités XAML) : Windows applique lui-même l'échelle, il n'y a rien à convertir. Les valeurs ont été arrondies à des multiples de 4 DIP (= 5 px à 125 %) pour garder des bords nets.

## Variante Seelen UI

Seelen UI (repo eythaann/Seelen-UI, v2.8.11, commit 2f9aaa3 of 2026-10-27, cloned and read directly) is deliberately FLAT: documentation/design-system.md forbids gradients, glassmorphism, borders and blur in the default theme. The only translucency allowed is a semi-transparent flat color on the toolbar (bg at 0.9 alpha), the dock (0.8), the dock window-preview (0.8) and the WM stack bar (0.9); every other surface (popovers, menus, start menu, tooltips, notifications) is opaque, radius 10 (--radius-m) or 16 (--radius-l), with soft shadows. Colors come from an oklch gray scale that inverts between dark and light (dark bg #181818, light bg #F2F2F2) plus the WINDOWS ACCENT for every highlight (hover = accent at 20 %, pressed 30 %, selected 10 %). Font is system-ui (Segoe UI Variable on Win11), weight 500 on the bar, 600 for titles. Defaults from libs/core/src/state/settings/mod.rs: Fancy Toolbar item 16 px + padding 8 + margin 0 = 32 px tall, position Top, never hidden; dock ('weg') item 40 px, padding 8, margin 8, gap 8 (72 px total), bottom, min-content (centered pill), hide-on-overlap, container radius 16, item radius 25 % of size (10 px), NO magnification (hover darkens the tile and brightens the icon 1.05; press = translateY 2 px). Default toolbar modules (src/background/state/application/toolbar_items.rs): LEFT user avatar+name, '|', focused-app icon+name, '-', window title; CENTER date (moment format 'ddd D MMM, hh:mm A' -> 'Wed 26 Jun, 05:13 PM'); RIGHT system tray, keyboard/lang, bluetooth, network, volume, power/battery, notifications, quick-settings. The bundled 'Bubbles' theme (src/static/themes/bubbles) and documentation/images/preview.png show the signature pill-groups look: transparent bar, each module its own rounded pill (bg at 0.95 alpha, radius 10 px). The old 'rofi for Windows' launcher (changelog, v1.x) no longer exists in 2.8: it was replaced by the opaque Apps Menu (start menu, radius 16, 7-column grid) and the Task Switcher. For the Windhawk variant: bar at the top (Taskbar on top mod v1.3 or the native position option in 26300.8493+ builds), DockHeight 40 (Seelen's 32 px needs the Taskbar height and icon size mod at 32/16), no gaps, transparent bar, pills 0.95 radius 10, hover/pressed = Windows accent at 20/30 %, opaque flyouts radius 10, no blur, no borders. All values are DIPs, so nothing changes at 125 % scale (the Seelen webview scales the same way).

Le bloc de constantes prêt à l'emploi est dans `variantes/seelen-toolbar.yaml`.

**Reproductible avec Windhawk :**
- Toolbar at the TOP of the screen: Windhawk mod 'Taskbar on top for Windows 11' (id taskbar-on-top, v1.3, by m417z; settings taskbarLocation=top, taskbarLocationSecondary, runningIndicatorsOnTop, startMenuAnimationAdjust, useNativeTaskbar; known limitation: with the non-native taskbar the Action Center Win+A stays at the bottom). Alternative: native position option (Insider build 26300.8493+ adds top/left/right + smaller buttons; reported for the Sept-2026 Patch Tuesday, GA not confirmed as of 2026-11-04). Then DockGapTop/DockGapBottom = 0 (Seelen margin 0) or 8/0 for a floating bar.
- Thin 32 px bar with 16 px icons (Seelen default): mod 'Taskbar height and icon size' set to height 32 / icon 16, native 'smaller taskbar buttons' on 26300.8493+; when combined, delete the Taskbar.TaskbarFrame Height=$DockHeight target and align DockHeight (README warning). Otherwise DockHeight=40 with TileInset=4 gives 32 px pills.
- Pill groups (Bubbles / preview.png look): already supported by the theme's tile targets; set DockBackground transparent, TileBackground/AppTileBackground = bg at 0.95, TileRadius=10, TileGap=2, TileInset=4, all borders Transparent. Flat default-theme look instead: DockBackground = bg at 0.90 with BlurAmount 0, tiles transparent.
- No blur / no borders / flat semi-transparent surfaces: WindhawkBlur with BlurAmount=0 or plain SolidColorBrush with Opacity 0.9 (bar) / 0.8 (dock) / 0.95 (pills); DockBorder, TileBorder, AppTileBorder = Transparent.
- Dark/light palette and Windows-accent following: themeResourceVariables Key@Dark/Key@Light (#181818 / #F2F2F2, tiles #1F1F1F / #FCFCFC) and {ThemeResource SystemAccentColor}, SystemAccentColorLight2 (dark) / SystemAccentColorDark1 (light) exactly like Seelen's --slu-std-ui-color = light-dark(accent-dark, accent-light).
- Accent-tinted hover/pressed/selected states (Seelen's signature): HoverBackground = SystemAccentColor at 0.20, PressedBackground 0.30, ActiveBackground 0.10 (applies to taskbar tiles, tray buttons, and via the Start Menu / Notification Center Styler files to menus, calendar days, quick settings).
- Running indicator like the Seelen dock: RunningIndicator target Width=3 Height=3 RadiusX/Y=1.5 Fill gray (#A4A4A4) or accent, Width@ActiveRunningIndicator=16 (50 % of tile) Fill accent, Fill@RequestingAttentionRunningIndicator accent, DotOffset=3; Border#MultiWindowElement collapsed (Seelen shows an instance-counter badge instead, not reproducible).
- Date/clock module identical to Seelen's center item ('Wed 26 Jun, 05:13 PM'): Taskbar Clock Customization v1.8.1 with TimeFormat 'hh:mm tt', DateFormat 'ddd d MMM', TopLine '%date%, %time%', BottomLine '' (single line), FontSize 13 Medium; French: 'ddd d MMM' + 'HH:mm'.
- System/media modules via Taskbar Clock Customization patterns: %time%, %date%, %weekday%, %weeknum%, %timezone%; %cpu%, %ram%, %ram_used%, %ram_total%, %gpu%, %vram%, %cpu_temp%, %gpu_temp%; %battery%, %battery_time%, %power%; %upload_speed%, %download_speed%, %disk_read%, %disk_write%; %media_info%, %media_title%, %media_artist%, %media_album%, %media_status%; %weather% (wttr.in, WebContentWeatherLocation), %web<n>%; %newline%. Seelen cpu plugin = 'CPU %cpu%', memory plugin = '%ram_used% / %ram_total% GB'. Put them on BottomLine or in TooltipLine (already done in lumabar-clock-widget.yaml).
- Dock hover magnification (NOT a Seelen default, but 'Apple Dock'-style community themes exist): mod 'Taskbar Dock Animation' v1.9.2 by Ph0en1x-dev (AnimationType 0 cosine, MaxScale 130 max with small taskbars, EffectRadius 100, SpacingFactor 50, BounceDelay, FocusDuration 150, MirrorForTopTaskbar=true when the bar is on top, DisableVerticalBounce).
- Opaque popups/menus/tooltips/thumbnails with radius 10 and no blur (Seelen .slu-std-popover, flyouts, context-menu: padding 8, min-width 200, item padding 8, labels 600): FlyoutBackground = solid #181818/#F2F2F2, FlyoutRadius=10, same in lumabar-notification-center-styler.yaml (PanelRadius 10-16, TileRadius 10, ChipRadius 6, hover accent 0.2) and lumabar-start-menu-styler.yaml (Apps Menu: PanelRadius 16, opaque, search input at top, 7-column grid, app icons radius 18 %, names 0.8rem/600, 24 px round avatar in footer; Phone Link side panel gets the same opaque radius-16 surface).
- Hide-on-overlap behavior (Seelen dock default): approximated with the Windhawk mod 'Taskbar auto-hide when maximized' (hide only when a window is maximized), not identical to Seelen's per-overlap detection.
- Start button as the Windows logo (Seelen weg_start_menu plugin): keep the native Start button tile; Seelen hides nothing there, so leave the //Visibility=Collapsed line commented.
- Badges on items (notification count, accent circle): native Windows taskbar badges already exist for Store apps; tray hover radius 8 maps to Seelen's radius-s/xs.
- Separators between groups (1 px muted line): approximate with the TileGap between pills; a real 1 px separator would need an extra Border target (not in the theme).

**Hors de portée (barre des tâches native) :**
- Tiling window manager (3 px accent borders, 10 px gaps/padding, bspwm/columns/grid layouts, stack bar with tabs, focus-follows animations): outside the taskbar; use FancyZones, komorebi or GlazeWM.
- A true second bar: Seelen runs the toolbar (top, 32 px) AND the dock (bottom, 72 px) at once; Windows has a single taskbar, so choose top toolbar OR bottom dock.
- Centered min-content dock pill (Seelen weg default) with the lumabar theme: the theme is full-width by design; a pill dock needs to start from the Styler's built-in LiquidGlass/Pills/OS26 themes and reuse these color constants.
- Workspaces module (dotted/named/numbered virtual-desktop indicator in the bar center) and the workspaces-viewer overlay: no Windhawk taskbar equivalent.
- User-menu module (avatar + display name) and focused-app module (icon + app name + '-' + window title) on the left: not exposed by Taskbar Styler or the Clock mod; only partial approximation is 'Combine taskbar buttons: Never' to show window titles on buttons.
- Seelen plugin system (JS templates, remote data, custom icons from react-icons, per-item style) and third-party widgets (media player, waveform, weather dashboards, clipboard): not reproducible.
- Rofi-style launcher: no longer exists in Seelen 2.8 (replaced by Apps Menu / Task Switcher); the Windows Start menu can only be re-skinned (Start Menu Styler), not replaced by a keyboard launcher (use PowerToys Run / Flow Launcher instead).
- Dock item press 'translateY(2px)', icon brightness/saturate filters on hover, mount/unmount scale 0.4->1 animations, 200 ms width transitions for medium/large items, media-session dock item: not styleable in XAML through the Styler.
- Instance counter badge (number of windows) bottom-right of a dock item: Windows only has the multi-window indicator, which this variant hides.
- Opaque-by-token contrast-adaptive oklch palette and translucency 'from bg color': only approximated with fixed @Dark/@Light hex values.
- Seelen's hide mode OnOverlap, per-monitor item visibility and reveal-on-attention UntilActivated: Windows auto-hide is all-or-nothing (partial approximation via 'Taskbar auto-hide when maximized').
- Dock scroll with inset edge shadows and horizontal overflow: the Windows taskbar overflows into a flyout instead.
- Box-shadows on tiles (--shadow-s) and the dock preview cards (256 px, radius 10, 16:9 thumbnail): XAML taskbar elements have no shadow property exposed by the Styler; thumbnail size is fixed by Windows.

## Mods essentiels

### Taskbar on top for Windows 11 (`taskbar-on-top`)

Imite : both. Barre d'outils en haut de l'écran (Seelen UI toolbar) / position top de lumabar

https://windhawk.net/mods/taskbar-on-top

Attention : Avec l'implémentation non native (anciens builds), le Centre d'action (Win+A) reste en bas : utiliser notification-center-follows-taskbar ou la règle Notification Center Styler indiquée dans le readme (issue #1053). startMenuAnimationAdjust peut ne pas fonctionner avec le panneau Phone Link ni avec certains thèmes Start Menu Styler (à vérifier par l'agent Démarrer). Si useNativeTaskbar=true sur un build 26H2+ la position Windows est utilisée ; sinon laisser Windows sur Bas. Pour imiter lumabar en bas : taskbarLocation: bottom ou ne pas installer. Aucun réglage DPI (125 % géré nativement).

```yaml
- taskbarLocation: top
- taskbarLocationSecondary: sameAsPrimary
- runningIndicatorsOnTop: false
- startMenuAnimationAdjust: true
- useNativeTaskbar: true
```

### Taskbar Dock Animation (`taskbar-dock-animation`)

Imite : seelen. Dock macOS de Seelen UI : grossissement des icônes au survol

https://windhawk.net/mods/taskbar-dock-animation

Attention : Readme : avec un mod/thème qui fixe la hauteur de la barre (style Taskbar.TaskbarFrame Height), garder MaxScale <= 130 sinon les icônes sont rognées ; icônes agrandies légèrement floues. MirrorForTopTaskbar: true uniquement si la barre est en haut (sinon false). TaskbarLabelsMode: true si Taskbar Labels affiche les titres. À 125 % de DPI, EffectRadius est en px : ajuster ~90-110. Incompatible StartAllBack. Ne pas cumuler avec taskbar-dock-animation-plus ni macos-style-dock.

```yaml
- AnimationType: 0
- MaxScale: 125
- EffectRadius: 90
- SpacingFactor: 35
- BounceDelay: 500
- FocusDuration: 150
- MirrorForTopTaskbar: true
- DisableVerticalBounce: false
- TaskbarLabelsMode: false
- ExcludeSystemButtonsMode: 2
- LerpSpeed: 60
- DisableBounce: true
```

### Translucent Windows (`translucent-windows`)

Imite : seelen. Fenêtres acryliques/translucides de Seelen UI (panneaux flottants translucides)

https://windhawk.net/mods/translucent-windows

Attention : acrylicsystem (vrai Acrylic SystemBackdrop) exige Windows 11 22H2+ et suit le thème sombre ; pour une teinte sombre explicite utiliser type: acrylicblur avec AccentBlurBehind: 80202020 (ARGB). Prérequis : effets de transparence activés, économiseur d'énergie désactivé. N'affecte pas UWP/WinUI, Electron/Chromium, Qt. Pour l'Explorateur, le readme impose windows-11-file-explorer-styler + thème Translucent Explorer11. Le mod s'injecte dans tous les processus (@include *) : exclure jeux/apps sensibles via règles (RuledPrograms) plutôt que l'exclusion Windhawk si SysColors est activé. Fonctionne mieux en thème sombre.

```yaml
- RenderingMod:
  - ThemeBackground: TRUE
  - SysColors: FALSE
  - AccentColorControls: TRUE
- BackgroundEffects:
  - type: acrylicsystem
  - AccentBlurBehind: "80202020"
- FlyoutsEffects: TRUE
```

### Taskbar Thumbnail Size (`taskbar-thumbnail-size`)

Imite : seelen. Aperçus de fenêtres plus grands façon dock Seelen (hover previews)

https://windhawk.net/mods/taskbar-thumbnail-size

Attention : Mode pourcentage (180 % de la taille d'origine). Les clés min/max ne servent que si useAbsoluteSize: true. Aucun conflit connu avec un thème qui fixe la hauteur de la barre.

```yaml
- size: 180
- useAbsoluteSize: false
- minWidth: 180
- minHeight: 120
- maxWidth: 180
- maxHeight: 120
- preserveAspectRatio: true
```

### Taskbar tray icon spacing and grid (`taskbar-notification-icon-spacing`)

Imite : both. Zone système compacte comme les modules tray de Seelen / la barre fine de lumabar

https://windhawk.net/mods/taskbar-notification-icon-spacing

Attention : Windows 11 22H2+. notificationIconRows: 2 donne une grille mais exige une barre assez haute : avec un thème qui réduit la hauteur (Taskbar.TaskbarFrame Height) la 2e rangée est rognée, rester à 1.

```yaml
- notificationIconWidth: 24
- notificationIconRows: 1
- gridArrangement: rowFirstLeftToRight
- overflowIconWidth: 32
- overflowIconsPerRow: 5
```

### Taskbar tray system icon tweaks (`taskbar-tray-system-icon-tweaks`)

Imite : both. Tray épuré : masque le bouton « Afficher le bureau », la cloche inactive et les icônes parasites (lumabar n'a pas de bouton bureau)

https://windhawk.net/mods/taskbar-tray-system-icon-tweaks

Attention : showDesktopButtonWidth est borné 0..800 dans la source ; 0 masque le bouton Afficher le bureau. Si separate-system-tray-icons est installé, laisser hideVolume/Network/Battery à false (il remplace déjà le bouton groupé). Windows 11 uniquement.

```yaml
- hideVolumeIcon: false
- hideNetworkIcon: false
- hideBatteryIcon: false
- grayscaleBatteryIcon: true
- hideMicrophoneIcon: false
- hideGeolocationIcon: true
- hideStudioEffectsIcon: true
- hideRecallIcon: true
- hideLanguageBar: false
- hideLanguageSupplementaryIcons: true
- hideBellIcon: whenInactive
- showDesktopButtonWidth: 0
```

### Taskbar Clock Customization (`taskbar-clock-customization`)

Imite : both. Modules date/heure, météo, stats système (CPU/RAM/GPU/réseau/batterie) et média de lumabar et Seelen UI

https://windhawk.net/mods/taskbar-clock-customization

Attention : Configuration déjà existante : id listé seulement. Clés utiles vérifiées : TopLine/BottomLine/TooltipLine, Width/Height/MaxWidth, DataCollection (NetworkMetricsFormat, UpdateInterval), MediaPlayer (MediaInfoFormat), WebContentWeatherLocation/Format/Units, TimeStyle. Conflits : micrographics-taskbar-clock, winklok et taskbar-desktop-indicator réécrivent aussi le texte de l'horloge ; taskbar-system-info et net-speed-taskbar dupliquent les métriques. Compagnon optionnel : taskbar-clock-spacer (token %s% pour justifier les éléments dans une largeur fixe).

## Mods recommandés

### Taskbar Virtual Desktop Switcher (`taskbar-vd-switcher`)

Imite : seelen. Module « workspaces » de la toolbar Seelen UI : un bouton cliquable par bureau virtuel

https://windhawk.net/mods/taskbar-vd-switcher

Attention : Pour reproduire Seelen (workspaces à gauche près du Démarrer) utiliser Position: "leftOfStart" ou "rightOfStart" (marqués expérimentaux). Arrangement "auto" s'adapte à la hauteur de la barre : avec un thème qui réduit la hauteur, réduire ItemHeight (ex. 18). Les boutons exposent les états Checked/CheckedPointerOver pour Taskbar Styler. Tailles en px logiques (DPI 125 % géré).

```yaml
- Placement:
  - Position: "beforeIcons"
  - AllTaskbars: false
- Content:
  - LabelFormat: "symbol"
  - ActiveSymbol: "●"
  - InactiveSymbol: "○"
  - TaskViewButton: false
- Layout:
  - Arrangement: "auto"
  - FillOrder: "rows"
  - Justify: "center"
  - NewItems: "append"
- Size:
  - ItemWidth: 20
  - ItemHeight: 22
  - ItemSpacing: 2
- Surface:
  - CornerRadius: 10
  - Opacity: 100
- State:
  - ActiveBackgroundColor: "accent"
  - InactiveBackgroundColor: ""
  - ActiveBold: false
- Behavior:
  - HoverPreview: true
  - PreviewDelay: 400
  - HideWhenSingle: true
```

### Always show all taskbar tray icons (`taskbar-notification-icons-show-all`)

Imite : seelen. Seelen affiche toutes les icônes de la zone de notification sans chevron de débordement

https://windhawk.net/mods/taskbar-notification-icons-show-all

Attention : Windows 11 uniquement. Si on préfère un tray minimal façon lumabar, utiliser mode: hideAll et tray-hover-expand pour le débordement au survol.

```yaml
- mode: showAll
```

### Separate System Tray Icons (`separate-system-tray-icons`)

Imite : seelen. Modules réseau / Bluetooth / son / batterie séparés comme dans la toolbar Seelen UI

https://windhawk.net/mods/separate-system-tray-icons

Attention : Remplace le bouton groupé (omni) : les styles Taskbar Styler ciblant SystemTray.OmniButton ne s'appliquent plus (le readme documente ses propres cibles Taskbar Styler). Le scroll volume sur l'icône son doublonne taskbar-volume-control : régler ce dernier sur scrollArea: taskbarWithoutNotificationArea ou laisser volumeWheelStep "0". Laisser hideVolume/Network/Battery à false dans taskbar-tray-system-icon-tweaks.

```yaml
- visibility:
  - showSoundButton: true
  - showBluetoothButton: true
  - showNetworkButton: true
  - showControlCenterButton: true
  - showBatteryButton: true
- buttonOrder: [network, bluetooth, sound, battery, controlcenter]
- battery:
  - showPercentage: always
  - smallGlyph: false
  - energySaverLeafGlyph: true
- sound:
  - volumeWheelStep: "5"
  - soundClickAction: sound_output
  - middleClickAction: toggle_mute
- iconPadding:
  - horizontal: 2
  - vertical: 0
```

### Island Media Controls (`island-media-controls`)

Imite : both. Module média de Seelen UI / widget média de lumabar : pochette, titre, contrôles, lecteur dépliable en verre

https://windhawk.net/mods/island-media-controls

Attention : N'installer qu'un seul widget média (island-media-controls OU taskbar-fluent-media-player OU taskbar-music-lounge OU le %media_title% de Taskbar Clock Customization). AutoSizeToTaskbar: true suit la hauteur imposée par le thème. Windhawk 2.0 recommandé pour l'interface de réglages. Material "liquid_glass" pour un rendu plus proche de Seelen, "transparent_borderless" pour lumabar.

```yaml
- Main:
  - DisplayMode: "compact"
  - Position: "tray_left"
  - Material: "acrylic"
  - ArtworkAbstractMode: "browser_original"
  - PopupButtonStyle: "solid_material"
  - PopupBackdropCoverEffect: "dark_only"
  - CompactAlbumBackground: true
  - CompactIslandTint: false
- Behavior:
  - HideWhenNoMedia: true
  - FeedbackLighting: true
  - AnimationSpeed: 100
  - HoverScale: 104
- Advanced:
  - AutoSizeToTaskbar: true
  - CompactWidth: 169
  - Height: 40
  - MarginLeft: 4
  - MarginRight: 4
  - ExpandedCornerRadius: 24
```

### Taskbar Elastic WinUI Pill (`taskbar-elastic-pill`)

Imite : seelen. Indicateur d'app active animé (pilule glissante) comme le dock Seelen UI

https://windhawk.net/mods/taskbar-elastic-pill

Attention : Margins = (horizontal, bas) : avec la barre en haut ou une hauteur imposée par le thème, vérifier visuellement et ajuster la marge basse. Conflit avec les styles du thème qui redessinent le running indicator et avec taskbar-count-badges en position bottom. Alternative : taskbar-blob-shape.

```yaml
- Pill:
  - Dimensions: "16, 3"
  - Margins: "0, 5"
  - PillRadius: '2'
  - HideInactiveDots: false
  - TrackSystemButtons: false
- Animation:
  - AnimationStyle: stretch
  - UseSpringPhysics: false
  - ElasticIntensity: '1.0'
  - SpeedMultiplier: '1.2'
  - FadeTransition: false
  - SquishTransition: false
- PointerInteractions:
  - HoverScale: '1.2, 1.0'
  - PressedScale: '0.85'
- Colors:
  - BgOpacity: '1.0, 1.0'
  - ColorMode: accent
  - CustomColor: ""
  - BgGradientDirection: horizontal
```

### Windows 11 File Explorer Styler (`windows-11-file-explorer-styler`)

Imite : seelen. Explorateur translucide (look fenêtres acryliques Seelen) — thèmes translucides intégrés : Translucent Explorer11, TintedGlass, LiquidGlass, WindowGlass, OS26 Liquid Glass, MicaBar, MicaTabless, Float

https://windhawk.net/mods/windows-11-file-explorer-styler

Attention : Le readme de Translucent Windows demande précisément ce mod + thème Translucent Explorer11 pour les parties WinUI de l'Explorateur. backgroundTranslucentEffect accepte acrylicblur/acrylic/mica/micaAlt si le thème ne le fixe pas. Un seul consommateur XAML diagnostics : conflit avec ExplorerBlurMica.

```yaml
- theme: "Translucent Explorer11"
- backgroundTranslucentEffect: ""
- backgroundTranslucentEffectRegion: ""
- styleConstants: [""]
- themeResourceVariables: [""]
- explorerFrameContainerHeight: 0
- xamlDiagnosticsHandling: alert
```

### Taskbar auto-hide when maximized (`taskbar-auto-hide-when-maximized`)

Imite : both. Dock flottant qui s'efface seulement sous une fenêtre maximisée (comportement lumabar/Seelen)

https://windhawk.net/mods/taskbar-auto-hide-when-maximized

Attention : Exige l'option Windows « Masquer automatiquement la barre des tâches » activée. mode: intersected si le thème flottant laisse des fenêtres passer sous la barre. Combiner avec taskbar-autohide-instant-show.

```yaml
- mode: maximized
- foregroundWindowOnly: false
- excludedPrograms: [""]
- primaryMonitorOnly: false
- oldTaskbarOnWin11: false
```

### Taskbar Auto-Hide Instant Show (`taskbar-autohide-instant-show`)

Imite : both. Apparition instantanée et animée (slide+fade) du dock, comme les animations de Seelen

https://windhawk.net/mods/taskbar-autohide-instant-show

Attention : Ne pas cumuler avec taskbar-auto-hide-speed (même hook). frameRate: 120 sur écran haute fréquence. edgeDetection: true si la barre n'apparaît pas sur un moniteur vide.

```yaml
- animationType: slideFade
- showSpeedup: 400
- hideSpeedup: 400
- showDuration: 200
- hideDuration: 200
- frameRate: 60
- unhideDelay: 1
- hideDelay: 150
- oldTaskbarOnWin11: false
- edgeDetection: false
```

### Taskbar Background Helper (`taskbar-background-helper`)

Imite : both. Remplit les parties transparentes de la barre (acrylique sombre) quand une fenêtre est maximisée — conçu pour Taskbar Styler

https://windhawk.net/mods/taskbar-background-helper

Attention : Avec un thème flottant, le remplissage couvre toute la largeur de la barre lorsqu'une fenêtre est maximisée (aspect « barre pleine »). Choisir entre ce mod et dynamic-taskbar-transparency (les deux écrivent le fond). Sert aussi de contournement à la limitation Taskbar Styler issue #742 (multi-barres).

```yaml
- backgroundStyle: acrylicBlur
- color:
  - red: 32
  - green: 32
  - blue: 32
  - accentColor: false
  - transparency: 128
- onlyWhenMaximized: true
- excludedPrograms: [""]
- styleForDarkMode:
  - use: false
```

### Taskbar Volume Control (`taskbar-volume-control`)

Imite : seelen. Volume à la molette sur la zone système, comme le module volume de Seelen

https://windhawk.net/mods/taskbar-volume-control

Attention : scrollArea limité au tray pour ne pas entrer en conflit avec taskbar-wheel-cycle (liste des tâches) et taskbar-scroll-actions. Les zones rendues click-through par le thème (IsHitTestVisible=False) ne reçoivent pas la molette. Si separate-system-tray-icons gère déjà la molette sur l'icône son, régler scrollArea: taskbarWithoutNotificationArea.

```yaml
- volumeIndicator: win11
- scrollArea: notification_area
- additionalScrollRegions: ""
- middleClickToMute: true
- ctrlScrollVolumeChange: false
- scrollAnywhereKeys:
  - shift: false
  - ctrl: false
  - alt: false
  - win: false
- fullScreenScrolling: disabled
- noAutomaticMuteToggle: false
- volumeChangeStep: 2
- oldTaskbarOnWin11: false
```

### Notification Center Follows Taskbar (`notification-center-follows-taskbar`)

Imite : seelen. Le centre de notifications (Win+N) s'ancre à la barre en haut, comme les panneaux flottants de Seelen

https://windhawk.net/mods/notification-center-follows-taskbar

Attention : À installer avec taskbar-on-top. Mettre repairHorizontalPosition: false si shell-flyout-positions est utilisé. Les règles de l'agent Notification Center Styler ne doivent pas déjà permuter/forcer la hauteur des panneaux si swapPanels est activé. Fermer/rouvrir le centre après activation.

```yaml
- swapPanels: false
- swapMediaPanels: false
- repairHorizontalPosition: true
```

### Taskbar Top Window Guard (`taskbar-top-window-guard`)

Imite : seelen. Empêche les fenêtres de s'ouvrir sous la toolbar en haut (confort Seelen)

https://windhawk.net/mods/taskbar-top-window-guard

Attention : Utile uniquement avec la barre en haut (taskbar-on-top ou position native). Fenêtres maximisées/plein écran ignorées. Tourne dans un processus outil Windhawk dédié.

```yaml
- centerWindowsOnOpen: false
- centerVerticalOffset: 0
- centerAlwaysOnTopWindows: false
- extraTopPadding: 0
- recheckDurationMs: 1500
- recheckIntervalMs: 100
```

## Mods optionnels

| Mod | Imite | Usage |
| --- | --- | --- |
| [Taskbar Dock Animation Plus](https://windhawk.net/mods/taskbar-dock-animation-plus) | seelen | Alternative au Dock Animation : même effet macOS avec détection automatique de la position (haut/bas/côtés) et correctifs multi-écrans |
| [Taskbar Labels for Windows 11](https://windhawk.net/mods/taskbar-labels) | both | Dock icônes seules avec indicateur centré dynamique (dock Seelen / lumabar) |
| [Cycle taskbar buttons with mouse wheel](https://windhawk.net/mods/taskbar-wheel-cycle) | seelen | Molette sur le dock pour passer d'une app à l'autre (confort dock Seelen) |
| [Click on empty taskbar space](https://windhawk.net/mods/taskbar-empty-space-clicks) | seelen | Actions sur l'espace vide (afficher le bureau, bureau virtuel suivant/précédent, gestionnaire de tâches) comme les raccourcis de la toolbar Seelen |
| [Middle click to close on the taskbar](https://windhawk.net/mods/taskbar-button-click) | both | Clic molette = fermer l'app (gestion type dock) |
| [Show all apps by default in start menu](https://windhawk.net/mods/start-menu-all-apps) | seelen | Lanceur Seelen : liste de toutes les apps dès l'ouverture |
| [Taskbar Desktop Indicator](https://windhawk.net/mods/taskbar-desktop-indicator) | seelen | Alternative légère au module workspaces : marqueurs ● ○ du bureau virtuel courant dans l'horloge |
| [Tray hover expand](https://windhawk.net/mods/tray-hover-expand) | lumabar | Débordement du tray ouvert au survol (tray minimal façon lumabar) |
| [Taskbar tray auto-hide (show on hover)](https://windhawk.net/mods/taskbar-tray-show-on-hover) | lumabar | Tray estompé tant qu'il n'est pas survolé (barre épurée lumabar) |
| [Taskbar Shadow & Border](https://windhawk.net/mods/taskbar-shadow-border) | lumabar | Ombre portée « flottante » au-dessus de la barre (dock flottant lumabar) |
| [Taskbar System Info](https://windhawk.net/mods/taskbar-system-info) | lumabar | Widget stats système de lumabar (CPU/GPU/RAM/VRAM, températures, graphes) |
| [Taskbar Network Speed Indicator](https://windhawk.net/mods/net-speed-taskbar) | lumabar | Widget débit réseau flottant avec sparkline (stats lumabar) |
| [Taskbar Fluent Media Player](https://windhawk.net/mods/taskbar-fluent-media-player) | both | Alternative au module média : lecteur Fluent intégré (acrylic/mica, pochette, boutons) |
| [Taskbar Music Lounge](https://windhawk.net/mods/taskbar-music-lounge) | lumabar | Alternative média minimaliste : ticker + pochette arrondie en acrylique |
| [Start button always on the left](https://windhawk.net/mods/taskbar-start-button-position) | seelen | Bouton lanceur à gauche + apps centrées (disposition toolbar Seelen) |
| [Dynamic Taskbar Transparency](https://windhawk.net/mods/dynamic-taskbar-transparency) | both | Machine à états type TranslucentTB : barre transparente sur le bureau, acrylique quand une fenêtre est maximisée ou un flyout ouvert |
| [Taskbar brightness and opacity tuner](https://windhawk.net/mods/taskbar-brightness-and-opacity-tuner) | lumabar | Masque la ligne supérieure et règle l'opacité fond/icônes (barre fine et sobre lumabar) |
| [Taskbar Margin](https://windhawk.net/mods/taskbar-margin) | seelen | Réserve une marge à gauche pour y loger un widget (stats/média) comme un module de toolbar |
| [Shell Flyout Positions](https://windhawk.net/mods/shell-flyout-positions) | seelen | Aligne les panneaux Notifications/Centre d'action sur la zone tray (panneaux flottants Seelen) |
| [Taskbar User Tile](https://windhawk.net/mods/taskbar-user-tile) | seelen | Module « user » (avatar + flyout compte) de la toolbar Seelen |
| [Hide Start Button](https://windhawk.net/mods/hide-start-button) | lumabar | Dock sans bouton Démarrer comme lumabar (menu via touche Win) |
| [Start Button Replacer](https://windhawk.net/mods/start-button-replacer) | seelen | Icône de lanceur personnalisée (logo custom de la toolbar Seelen) |
| [Start button colorizer](https://windhawk.net/mods/taskbar-start-button-colorizer) | seelen | Logo Démarrer monochrome/accent et redimensionné (icône lanceur sobre Seelen) |
| [Window Border Customizer](https://windhawk.net/mods/window-border-customizer) | seelen | Bordures de fenêtres fines et translucides (fenêtres Seelen) |
| [Translucent Flyouts Controller](https://windhawk.net/mods/translucent-flyouts-controller) | seelen | Menus contextuels/tooltips acryliques arrondis (pilote TranslucentFlyouts) |
| [Taskbar Icon Animation](https://windhawk.net/mods/taskbar-hover-animations) | seelen | Rebond macOS au survol (overlay) en complément/alternative du grossissement |
| [Taskbar Count Badges](https://windhawk.net/mods/taskbar-count-badges) | seelen | Points par fenêtre ouverte sous l'icône (indicateur multi-fenêtres du dock Seelen) |
| [MacOS Style Dock](https://windhawk.net/mods/macos-style-dock) | seelen | Alternative tout-en-un : transforme la barre en dock de verre dépoli flottant avec magnification (look dock Seelen) |
| [Mac Island & Dock](https://windhawk.net/mods/mac-island-dock) | both | Îlot flottant en haut (horloge, centre de contrôle, notifications, média, icônes tray) — le plus proche d'un dock-widget lumabar / d'une toolbar Seelen, en gardant notre barre intacte |
| [Dynamic Island for Windows](https://windhawk.net/mods/dynamic-island-for-windows) | lumabar | Pilule flottante (média, météo, calendrier, batterie, stats matériel) = widget dock lumabar sur le bureau |
| [Taskbar Fade](https://windhawk.net/mods/taskbar-fade) | lumabar | Barre qui s'estompe quand la souris s'éloigne (dock discret lumabar) |
| [Taskbar Blob Shape](https://windhawk.net/mods/taskbar-blob-shape) | seelen | Alternative à la pilule : forme « blob » derrière l'app active (thème Blob du guide Taskbar Styler) |
| [Taskbar Separators](https://windhawk.net/mods/taskbar-separators) | seelen | Séparateurs visuels entre groupes d'icônes (séparateurs du dock Seelen) |
| [Disable grouping on the taskbar](https://windhawk.net/mods/taskbar-grouping) | seelen | Un item par fenêtre avec épinglés fixes (comportement dock Seelen par fenêtre) |
| [Tray Overflow List](https://windhawk.net/mods/tray-overflow-list) | both | Flyout de débordement en liste avec noms d'apps (tray lisible) |
| [Taskbar Scroll Actions](https://windhawk.net/mods/taskbar-scroll-actions) | seelen | Molette sur une zone de la barre pour changer de bureau virtuel (workspaces Seelen) |
| [GNOME-like dynamic virtual desktops](https://windhawk.net/mods/gnome-dynamic-desktops) | seelen | Workspaces dynamiques (toujours un bureau vide en fin de liste) comme les workspaces de Seelen |
| [Live Update for Windows 11](https://windhawk.net/mods/live-update-for-windows-widget) | lumabar | Slot « Live Activity » unique (média, batterie, minuteurs, hotspot, confidentialité) intégré à la barre — widget contextuel façon lumabar |
| [Instant Taskbar Thumbnail Previews](https://windhawk.net/mods/instant-taskbar-thumbnail-previews) | seelen | Aperçus de fenêtres rapides et ergonomiques (hover previews du dock Seelen) |
| [Taskbar Volume Control Per-App](https://windhawk.net/mods/taskbar-volume-control-per-app) | seelen | Volume par application depuis l'icône du dock (mixeur du module média Seelen) |
| [Better Volume Mixer](https://windhawk.net/mods/better-volume-mixer) | seelen | Mixeur compact en verre dépoli depuis le tray (panneau son flottant Seelen) |
| [Desktop Icon Dock](https://windhawk.net/mods/desktop-icon-dock) | both | Dock macOS autonome sur le bureau (panneau Liquid Glass, magnification) pour un vrai dock flottant à côté de la toolbar |
| [Taskbar Clock to Left](https://windhawk.net/mods/taskbar-clock-to-left) | seelen | Horloge native déplacée à gauche (disposition de modules personnalisée de la toolbar Seelen) |

## Notes de l'inventaire

Méthode : chaque id a été vérifié dans le catalogue officiel Windhawk (https://mods.windhawk.net/catalog.json, 860 mods, v1.7.3) et la page windhawk.net/mods/<id> (HTTP 200 testé) ; la source de chacun des 84 candidats a été lue depuis raw.githubusercontent.com/ramensoftware/windhawk-mods/main/mods/<id>.wh.cpp (blocs ==WindhawkModReadme== et ==WindhawkModSettings==). Toutes les clés/valeurs des YAML proviennent de ces blocs ; en mode Textual, les clés omises gardent leur valeur par défaut. Exclus sur consigne : windows-11-taskbar-styler, taskbar-icon-size, windows-11-start-menu-styler (y compris ses options de layout), windows-11-notification-center-styler.

Candidats inexistants ou écartés : aucun mod « click-through taskbar » n'existe dans le catalogue (c'est un comportement de thème Taskbar Styler : fond/espaces rendus IsHitTestVisible=False) ; « Taskbar auto-hide speed » existe (taskbar-auto-hide-speed) mais fait doublon avec taskbar-autohide-instant-show, choisir un seul ; acrylic-effect-radius-changer existe mais est Windows 10 uniquement (injection DWM) → écarté ; aero-tray exige l'injection dans winlogon.exe → écarté ; micrographics-taskbar-clock et winklok remplacent l'horloge et entrent en conflit avec Taskbar Clock Customization → écartés ; taskbar-show-desktop-button-aero-peek fait l'inverse de ce qu'on veut (on masque le bouton via showDesktopButtonWidth: 0) ; taskbar-z-order-override, taskbar-multi-tray, virtual-desktop-helper, keyboard-shortcut-actions, task-view-customizer, taskbar-icon-group-centering, taskbar-thumbnail-reorder, taskbar-left-click-cycle : existants mais sans apport visuel lumabar/Seelen, non listés.

Conflits avec un thème qui fixe la hauteur de la barre (Taskbar.TaskbarFrame Height) : taskbar-dock-animation(-plus) → MaxScale <= 130 (rognage) ; taskbar-notification-icon-spacing → rows: 1 seulement ; taskbar-vd-switcher → Arrangement auto s'adapte, réduire ItemHeight ; taskbar-system-info et island-media-controls (AutoSizeToTaskbar) s'adaptent ; taskbar-fluent-media-player (playerHeight), taskbar-music-lounge (PanelHeight), taskbar-separators (height), taskbar-user-tile (pictureSize), start-button-replacer (iconSize) ont des hauteurs fixes à aligner ; taskbar-elastic-pill/taskbar-blob-shape/taskbar-count-badges ont des marges/offsets à revérifier ; macos-style-dock impose sa propre hauteur → incompatible avec le thème.

Conflits avec un thème click-through : taskbar-empty-space-clicks, taskbar-volume-control, taskbar-wheel-cycle, taskbar-scroll-actions, taskbar-collapse-to-running ne reçoivent ni clics ni molette sur les zones click-through.

Groupes « n'en installer qu'un » : grossissement (taskbar-dock-animation | taskbar-dock-animation-plus | macos-style-dock | taskbar-hover-animations) ; fond de barre (taskbar-background-helper | dynamic-taskbar-transparency, jamais avec TranslucentTB) ; widget média (island-media-controls | taskbar-fluent-media-player | taskbar-music-lounge | live-update-for-windows-widget | %media% de Taskbar Clock Customization) ; indicateur actif (taskbar-elastic-pill | taskbar-blob-shape | count-badges bottom) ; workspaces (taskbar-vd-switcher | taskbar-desktop-indicator) ; animation auto-hide (taskbar-autohide-instant-show | taskbar-auto-hide-speed) ; position des flyouts (shell-flyout-positions | repairHorizontalPosition de notification-center-follows-taskbar) ; menus translucides (FlyoutsEffects de translucent-windows | translucent-flyouts-controller) ; bouton Démarrer (hide-start-button | start-button-replacer | taskbar-start-button-colorizer).

DPI 125 % : aucun mod n'a de réglage DPI dédié ; les tailles en px logiques (vd-switcher, system-info, margin avec followDpi: true, dynamic-island AutoDpiScale) sont mises à l'échelle automatiquement ; EffectRadius du Dock Animation et les % de position (net-speed) sont à ajuster à l'œil.

Démarrer / Phone Link : taskbar-on-top indique que startMenuAnimationAdjust peut ne pas fonctionner avec le panneau Phone Link ni avec certains thèmes Start Menu Styler ; start-menu-all-apps et startMenu de shell-flyout-positions sont à valider par l'agent qui gère le menu Démarrer. Le centre de notifications en haut relève de l'agent Notification Center Styler (règle issue #1053) ; notification-center-follows-taskbar en est le complément côté barre.

Scénario Seelen (barre en haut) : taskbar-on-top + dock animation + vd-switcher + separate-system-tray-icons + island-media-controls + translucent-windows + file-explorer-styler + notification-center-follows-taskbar + top-window-guard. Scénario lumabar (barre en bas) : ne pas installer taskbar-on-top ; ajouter taskbar-auto-hide-when-maximized + instant-show, et éventuellement mac-island-dock (îlot en haut, taskbarMode: windows) ou dynamic-island-for-windows pour le widget flottant.
