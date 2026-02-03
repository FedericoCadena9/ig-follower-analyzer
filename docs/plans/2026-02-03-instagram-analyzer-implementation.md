# Instagram Follower Analyzer - Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a Vue 3 + Tailwind single-page app that compares Instagram followers/following JSON exports to show accounts that don't follow back.

**Architecture:** Single HTML file with Vue 3 and Tailwind via CDN. No build tools. Components defined inline using Composition API. localStorage for whitelist persistence.

**Tech Stack:** Vue 3 (CDN), Tailwind CSS (CDN), vanilla JavaScript

---

## Task 1: Project Scaffold

**Files:**
- Create: `index.html`
- Create: `sample-data/followers.json`
- Create: `sample-data/following.json`
- Create: `sample-data/whitelist.json`

**Step 1: Create base HTML with Vue + Tailwind CDN**

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Instagram Follower Analyzer</title>
  <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-50 min-h-screen">
  <div id="app"></div>

  <script>
    const { createApp, ref, computed } = Vue;

    createApp({
      setup() {
        const message = ref('Instagram Follower Analyzer');
        return { message };
      },
      template: `
        <div class="max-w-4xl mx-auto px-4 py-8">
          <h1 class="text-2xl font-bold text-gray-800">{{ message }}</h1>
        </div>
      `
    }).mount('#app');
  </script>
</body>
</html>
```

**Step 2: Create sample followers.json (Format A)**

```json
[
  {
    "title": "user_one",
    "media_list_data": [],
    "string_list_data": [
      {
        "href": "https://www.instagram.com/user_one",
        "value": "user_one",
        "timestamp": 1704067200
      }
    ]
  },
  {
    "title": "user_two",
    "media_list_data": [],
    "string_list_data": [
      {
        "href": "https://www.instagram.com/user_two",
        "value": "user_two",
        "timestamp": 1704153600
      }
    ]
  },
  {
    "title": "mutual_friend",
    "media_list_data": [],
    "string_list_data": [
      {
        "href": "https://www.instagram.com/mutual_friend",
        "value": "mutual_friend",
        "timestamp": 1704240000
      }
    ]
  }
]
```

**Step 3: Create sample following.json (Format B)**

```json
{
  "relationships_following": [
    {
      "title": "mutual_friend",
      "string_list_data": [
        {
          "href": "https://www.instagram.com/mutual_friend",
          "value": "mutual_friend",
          "timestamp": 1704067200
        }
      ]
    },
    {
      "title": "not_following_back",
      "string_list_data": [
        {
          "href": "https://www.instagram.com/not_following_back",
          "value": "not_following_back",
          "timestamp": 1704153600
        }
      ]
    },
    {
      "title": "celebrity_page",
      "string_list_data": [
        {
          "href": "https://www.instagram.com/celebrity_page",
          "value": "celebrity_page",
          "timestamp": 1704240000
        }
      ]
    }
  ]
}
```

**Step 4: Create sample whitelist.json**

```json
["celebrity_page"]
```

**Step 5: Verify by opening index.html in browser**

Expected: Page shows "Instagram Follower Analyzer" heading with gray background.

**Step 6: Commit**

```bash
git add index.html sample-data/
git commit -m "feat: scaffold project with Vue 3 + Tailwind CDN and sample data"
```

---

## Task 2: File Upload Components (DropZone)

**Files:**
- Modify: `index.html`

**Step 1: Add DropZone component and file upload state**

Replace the script section in index.html:

```html
<script>
const { createApp, ref, computed, reactive } = Vue;

// DropZone Component
const DropZone = {
  props: {
    label: String,
    accept: { type: String, default: '.json' },
    loaded: Boolean,
    count: { type: Number, default: null },
    error: { type: String, default: null }
  },
  emits: ['file-loaded'],
  setup(props, { emit }) {
    const isDragging = ref(false);
    const fileInput = ref(null);

    const handleDrop = (e) => {
      isDragging.value = false;
      const file = e.dataTransfer.files[0];
      if (file) processFile(file);
    };

    const handleClick = () => {
      fileInput.value?.click();
    };

    const handleFileSelect = (e) => {
      const file = e.target.files[0];
      if (file) processFile(file);
    };

    const processFile = (file) => {
      const reader = new FileReader();
      reader.onload = (e) => {
        try {
          const data = JSON.parse(e.target.result);
          emit('file-loaded', { data, filename: file.name });
        } catch (err) {
          emit('file-loaded', { error: 'JSON inválido: ' + err.message });
        }
      };
      reader.readAsText(file);
    };

    return { isDragging, fileInput, handleDrop, handleClick, handleFileSelect };
  },
  template: `
    <div
      @click="handleClick"
      @dragover.prevent="isDragging = true"
      @dragleave.prevent="isDragging = false"
      @drop.prevent="handleDrop"
      class="relative border-2 border-dashed rounded-xl p-6 text-center cursor-pointer transition-all min-h-[120px] flex flex-col items-center justify-center"
      :class="{
        'border-blue-400 bg-blue-50': isDragging,
        'border-green-400 bg-green-50': loaded && !error,
        'border-red-400 bg-red-50': error,
        'border-gray-300 bg-white hover:border-gray-400 hover:bg-gray-50': !isDragging && !loaded && !error
      }"
    >
      <input
        ref="fileInput"
        type="file"
        :accept="accept"
        @change="handleFileSelect"
        class="hidden"
      />

      <div v-if="error" class="text-red-600">
        <span class="text-2xl">❌</span>
        <p class="mt-2 font-medium">Error</p>
        <p class="text-sm">{{ error }}</p>
      </div>

      <div v-else-if="loaded" class="text-green-600">
        <span class="text-2xl">✅</span>
        <p class="mt-2 font-medium">{{ label }}</p>
        <p class="text-sm text-green-700" v-if="count !== null">{{ count.toLocaleString() }} registros</p>
      </div>

      <div v-else class="text-gray-500">
        <span class="text-2xl">📁</span>
        <p class="mt-2 font-medium">{{ label }}</p>
        <p class="text-sm">Arrastra o haz clic</p>
      </div>
    </div>
  `
};

createApp({
  components: { DropZone },
  setup() {
    const followers = ref(null);
    const following = ref(null);
    const whitelist = ref(null);
    const errors = reactive({
      followers: null,
      following: null,
      whitelist: null
    });

    const handleFollowersLoaded = ({ data, error, filename }) => {
      if (error) {
        errors.followers = error;
        followers.value = null;
      } else {
        errors.followers = null;
        followers.value = data;
      }
    };

    const handleFollowingLoaded = ({ data, error, filename }) => {
      if (error) {
        errors.following = error;
        following.value = null;
      } else {
        errors.following = null;
        following.value = data;
      }
    };

    const handleWhitelistLoaded = ({ data, error, filename }) => {
      if (error) {
        errors.whitelist = error;
        whitelist.value = null;
      } else {
        errors.whitelist = null;
        whitelist.value = data;
      }
    };

    const followersCount = computed(() => {
      if (!followers.value) return null;
      return Array.isArray(followers.value) ? followers.value.length : 0;
    });

    const followingCount = computed(() => {
      if (!following.value) return null;
      const data = following.value.relationships_following || following.value;
      return Array.isArray(data) ? data.length : 0;
    });

    return {
      followers,
      following,
      whitelist,
      errors,
      followersCount,
      followingCount,
      handleFollowersLoaded,
      handleFollowingLoaded,
      handleWhitelistLoaded
    };
  },
  template: `
    <div class="max-w-4xl mx-auto px-4 py-8">
      <header class="text-center mb-8">
        <h1 class="text-2xl md:text-3xl font-bold text-gray-800">📊 Instagram Follower Analyzer</h1>
        <p class="text-gray-600 mt-2">Descubre quién no te sigue de vuelta</p>
      </header>

      <section class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-8">
        <DropZone
          label="Followers"
          :loaded="!!followers"
          :count="followersCount"
          :error="errors.followers"
          @file-loaded="handleFollowersLoaded"
        />
        <DropZone
          label="Following"
          :loaded="!!following"
          :count="followingCount"
          :error="errors.following"
          @file-loaded="handleFollowingLoaded"
        />
        <DropZone
          label="Whitelist (opcional)"
          :loaded="!!whitelist"
          :count="Array.isArray(whitelist) ? whitelist.length : whitelist?.usernames?.length"
          :error="errors.whitelist"
          @file-loaded="handleWhitelistLoaded"
        />
      </section>

      <div v-if="followers && following" class="text-center text-gray-600">
        <p>✅ Archivos cargados. Procesando...</p>
      </div>
    </div>
  `
}).mount('#app');
</script>
```

**Step 2: Test file upload with sample data**

1. Open index.html in browser
2. Drag sample-data/followers.json to first drop zone
3. Drag sample-data/following.json to second drop zone
4. Verify both show green checkmarks with record counts

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add DropZone component for file uploads"
```

---

## Task 3: JSON Parser & Normalizer

**Files:**
- Modify: `index.html`

**Step 1: Add normalizer functions and warnings system**

Add after the DropZone component definition:

```javascript
// Normalizer utility
const normalizeData = (data, type) => {
  const warnings = [];
  const normalized = new Map();

  // Detect format and extract array
  let items;
  if (Array.isArray(data)) {
    // Format A: direct array
    items = data;
  } else if (data.relationships_following) {
    // Format B: following with wrapper
    items = data.relationships_following;
  } else if (data.relationships_followers) {
    // Format B variant: followers with wrapper
    items = data.relationships_followers;
  } else {
    // Try to find any array property
    const arrayProp = Object.keys(data).find(k => Array.isArray(data[k]));
    if (arrayProp) {
      items = data[arrayProp];
      warnings.push({
        type: 'info',
        message: `Formato detectado: usando propiedad "${arrayProp}"`
      });
    } else {
      return { normalized, warnings: [{ type: 'error', message: 'No se encontró array de usuarios en el archivo' }] };
    }
  }

  // Check for empty data
  if (!items || items.length === 0) {
    return { normalized, warnings: [{ type: 'error', message: 'El archivo no contiene usuarios' }] };
  }

  // Check for outdated data (6 months = ~15768000 seconds)
  const sixMonthsAgo = Math.floor(Date.now() / 1000) - 15768000;
  let mostRecentTimestamp = 0;

  // Process each item
  const seenHrefs = new Map();

  for (const item of items) {
    const stringData = item.string_list_data?.[0];
    if (!stringData) {
      warnings.push({
        type: 'warning',
        message: `Usuario "${item.title || 'desconocido'}" no tiene datos válidos`
      });
      continue;
    }

    const { href, value, timestamp } = stringData;

    // Track most recent timestamp
    if (timestamp > mostRecentTimestamp) {
      mostRecentTimestamp = timestamp;
    }

    // Check for duplicates
    if (seenHrefs.has(href)) {
      const existing = seenHrefs.get(href);
      warnings.push({
        type: 'warning',
        message: `Duplicado: "${value}" aparece más de una vez`
      });
      // Keep the one with more recent timestamp
      if (timestamp > existing.timestamp) {
        seenHrefs.set(href, { username: value, href, timestamp });
      }
      continue;
    }

    // Check for valid Instagram URL
    if (!href?.includes('instagram.com/')) {
      warnings.push({
        type: 'warning',
        message: `URL inválida para "${value}": ${href}`
      });
    }

    seenHrefs.set(href, { username: value, href, timestamp });
  }

  // Check for outdated data
  if (mostRecentTimestamp > 0 && mostRecentTimestamp < sixMonthsAgo) {
    const date = new Date(mostRecentTimestamp * 1000).toLocaleDateString('es');
    warnings.push({
      type: 'info',
      message: `Datos desactualizados: último registro del ${date}. Considera descargar datos frescos.`
    });
  }

  // Convert to Map
  for (const [href, user] of seenHrefs) {
    normalized.set(href, user);
  }

  return { normalized, warnings };
};

// Compare followers and following
const compareUsers = (followersMap, followingMap, whitelistSet) => {
  const notFollowingBack = [];
  const warnings = [];

  for (const [href, user] of followingMap) {
    // Check if in whitelist (case insensitive)
    if (whitelistSet.has(user.username.toLowerCase())) {
      continue;
    }

    // Check if they follow back
    if (!followersMap.has(href)) {
      notFollowingBack.push(user);
    } else {
      // Check for username mismatch (username changed)
      const follower = followersMap.get(href);
      if (follower.username !== user.username) {
        warnings.push({
          type: 'warning',
          message: `Username cambiado: "${follower.username}" → "${user.username}" (mismo perfil)`
        });
      }
    }
  }

  // Sort by timestamp descending (most recent first)
  notFollowingBack.sort((a, b) => b.timestamp - a.timestamp);

  return { notFollowingBack, warnings };
};

// Parse whitelist (supports array or object format)
const parseWhitelist = (data) => {
  if (Array.isArray(data)) {
    return new Set(data.map(u => u.toLowerCase()));
  }
  if (data?.usernames && Array.isArray(data.usernames)) {
    return new Set(data.usernames.map(u => u.toLowerCase()));
  }
  return new Set();
};
```

**Step 2: Update the main app to use normalizer**

Update the setup() function:

```javascript
setup() {
  // ... existing refs ...

  const followersNormalized = ref(null);
  const followingNormalized = ref(null);
  const whitelistSet = ref(new Set());
  const allWarnings = ref([]);
  const results = ref([]);

  // Load whitelist from localStorage on mount
  const storedWhitelist = localStorage.getItem('ig-analyzer-whitelist');
  if (storedWhitelist) {
    try {
      const parsed = JSON.parse(storedWhitelist);
      whitelistSet.value = new Set(parsed.usernames?.map(u => u.toLowerCase()) || []);
    } catch (e) {
      console.warn('Could not parse stored whitelist');
    }
  }

  const processData = () => {
    if (!followers.value || !following.value) return;

    const warnings = [];

    // Normalize followers
    const { normalized: normFollowers, warnings: warnFollowers } = normalizeData(followers.value, 'followers');
    followersNormalized.value = normFollowers;
    warnings.push(...warnFollowers);

    // Normalize following
    const { normalized: normFollowing, warnings: warnFollowing } = normalizeData(following.value, 'following');
    followingNormalized.value = normFollowing;
    warnings.push(...warnFollowing);

    // Check for blocking errors
    const hasBlockingError = warnings.some(w => w.type === 'error');
    if (hasBlockingError) {
      allWarnings.value = warnings;
      results.value = [];
      return;
    }

    // Merge imported whitelist with stored whitelist
    if (whitelist.value) {
      const imported = parseWhitelist(whitelist.value);
      for (const username of imported) {
        whitelistSet.value.add(username);
      }
      saveWhitelist();
    }

    // Compare
    const { notFollowingBack, warnings: compareWarnings } = compareUsers(
      normFollowers,
      normFollowing,
      whitelistSet.value
    );

    warnings.push(...compareWarnings);
    allWarnings.value = warnings;
    results.value = notFollowingBack;
  };

  const saveWhitelist = () => {
    const data = {
      version: 1,
      updatedAt: new Date().toISOString(),
      usernames: Array.from(whitelistSet.value)
    };
    localStorage.setItem('ig-analyzer-whitelist', JSON.stringify(data));
  };

  // Watch for changes and process
  const handleFollowersLoaded = ({ data, error }) => {
    if (error) {
      errors.followers = error;
      followers.value = null;
    } else {
      errors.followers = null;
      followers.value = data;
      processData();
    }
  };

  const handleFollowingLoaded = ({ data, error }) => {
    if (error) {
      errors.following = error;
      following.value = null;
    } else {
      errors.following = null;
      following.value = data;
      processData();
    }
  };

  const handleWhitelistLoaded = ({ data, error }) => {
    if (error) {
      errors.whitelist = error;
      whitelist.value = null;
    } else {
      errors.whitelist = null;
      whitelist.value = data;
      processData();
    }
  };

  return {
    // ... existing returns ...
    allWarnings,
    results,
    whitelistSet,
    processData
  };
}
```

**Step 3: Test with sample data**

1. Open browser, load both sample files
2. Verify "mutual_friend" is NOT in results (they follow back)
3. Verify "not_following_back" IS in results
4. Verify "celebrity_page" is NOT in results (whitelisted)

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add JSON normalizer with edge case detection"
```

---

## Task 4: Warnings Panel Component

**Files:**
- Modify: `index.html`

**Step 1: Add WarningsPanel component**

```javascript
const WarningsPanel = {
  props: {
    warnings: { type: Array, default: () => [] }
  },
  setup(props) {
    const isExpanded = ref(true);

    const groupedWarnings = computed(() => {
      const groups = { error: [], warning: [], info: [] };
      for (const w of props.warnings) {
        groups[w.type]?.push(w);
      }
      return groups;
    });

    const hasWarnings = computed(() => props.warnings.length > 0);

    return { isExpanded, groupedWarnings, hasWarnings };
  },
  template: `
    <div v-if="hasWarnings" class="mb-6">
      <button
        @click="isExpanded = !isExpanded"
        class="w-full flex items-center justify-between p-4 bg-amber-50 border border-amber-200 rounded-lg hover:bg-amber-100 transition-colors"
      >
        <span class="flex items-center gap-2 font-medium text-amber-800">
          <span>⚠️</span>
          <span>{{ warnings.length }} problema{{ warnings.length !== 1 ? 's' : '' }} encontrado{{ warnings.length !== 1 ? 's' : '' }}</span>
        </span>
        <span class="text-amber-600">{{ isExpanded ? '▼' : '▶' }}</span>
      </button>

      <div v-show="isExpanded" class="mt-3 space-y-3">
        <!-- Errors -->
        <div
          v-for="(warning, idx) in groupedWarnings.error"
          :key="'error-' + idx"
          class="p-4 bg-red-50 border-l-4 border-red-500 rounded-r-lg"
        >
          <div class="flex items-start gap-2">
            <span class="text-red-500 font-bold">🔴 ERROR</span>
          </div>
          <p class="mt-1 text-red-800">{{ warning.message }}</p>
        </div>

        <!-- Warnings -->
        <div
          v-for="(warning, idx) in groupedWarnings.warning"
          :key="'warning-' + idx"
          class="p-4 bg-amber-50 border-l-4 border-amber-500 rounded-r-lg"
        >
          <div class="flex items-start gap-2">
            <span class="text-amber-600 font-bold">🟡 WARNING</span>
          </div>
          <p class="mt-1 text-amber-800">{{ warning.message }}</p>
        </div>

        <!-- Info -->
        <div
          v-for="(warning, idx) in groupedWarnings.info"
          :key="'info-' + idx"
          class="p-4 bg-blue-50 border-l-4 border-blue-500 rounded-r-lg"
        >
          <div class="flex items-start gap-2">
            <span class="text-blue-600 font-bold">🟠 INFO</span>
          </div>
          <p class="mt-1 text-blue-700">{{ warning.message }}</p>
          <p class="mt-2 text-sm text-blue-600">
            💡 Instagram → Configuración → Tu actividad → Descargar información
          </p>
        </div>
      </div>
    </div>
  `
};
```

**Step 2: Add to main app template**

```html
<WarningsPanel :warnings="allWarnings" />
```

**Step 3: Test with edge cases**

Create a malformed JSON file and verify error displays correctly.

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add WarningsPanel component with visual severity levels"
```

---

## Task 5: Results Table (Mobile Cards + Desktop Table)

**Files:**
- Modify: `index.html`

**Step 1: Add ResultsSection component**

```javascript
const ResultsSection = {
  props: {
    results: { type: Array, default: () => [] },
    whitelistSet: { type: Set, default: () => new Set() }
  },
  emits: ['add-to-whitelist'],
  setup(props, { emit }) {
    const searchQuery = ref('');
    const sortOrder = ref('newest');

    const filteredResults = computed(() => {
      let filtered = [...props.results];

      // Apply search filter
      if (searchQuery.value.trim()) {
        const query = searchQuery.value.toLowerCase();
        filtered = filtered.filter(u => u.username.toLowerCase().includes(query));
      }

      // Apply sort
      if (sortOrder.value === 'newest') {
        filtered.sort((a, b) => b.timestamp - a.timestamp);
      } else if (sortOrder.value === 'oldest') {
        filtered.sort((a, b) => a.timestamp - b.timestamp);
      } else if (sortOrder.value === 'alpha') {
        filtered.sort((a, b) => a.username.localeCompare(b.username));
      }

      return filtered;
    });

    const formatDate = (timestamp) => {
      return new Date(timestamp * 1000).toLocaleDateString('es', {
        day: 'numeric',
        month: 'short',
        year: 'numeric'
      });
    };

    const openProfile = (href) => {
      window.open(href, '_blank');
    };

    const addToWhitelist = (user) => {
      emit('add-to-whitelist', user.username);
    };

    return {
      searchQuery,
      sortOrder,
      filteredResults,
      formatDate,
      openProfile,
      addToWhitelist
    };
  },
  template: `
    <section v-if="results.length > 0">
      <!-- Search and Sort Bar -->
      <div class="flex flex-col sm:flex-row gap-3 mb-4">
        <div class="flex-1 relative">
          <span class="absolute left-3 top-1/2 -translate-y-1/2 text-gray-400">🔍</span>
          <input
            v-model="searchQuery"
            type="text"
            placeholder="Buscar username..."
            class="w-full pl-10 pr-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none"
          />
        </div>
        <select
          v-model="sortOrder"
          class="px-4 py-3 border border-gray-300 rounded-lg bg-white focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none"
        >
          <option value="newest">Más recientes</option>
          <option value="oldest">Más antiguos</option>
          <option value="alpha">A-Z</option>
        </select>
      </div>

      <!-- Results Count -->
      <p class="text-gray-600 mb-4">
        📋 <strong>{{ filteredResults.length }}</strong> de {{ results.length }} no te siguen
      </p>

      <!-- Mobile Cards (visible on small screens) -->
      <div class="md:hidden space-y-3">
        <div
          v-for="user in filteredResults"
          :key="user.href"
          class="bg-white border border-gray-200 rounded-xl p-4 shadow-sm"
        >
          <div class="flex items-center justify-between mb-2">
            <span class="font-semibold text-gray-800">@{{ user.username }}</span>
          </div>
          <p class="text-sm text-gray-500 mb-3">Seguido: {{ formatDate(user.timestamp) }}</p>
          <div class="flex gap-2">
            <button
              @click="openProfile(user.href)"
              class="flex-1 min-h-[44px] px-4 py-2 bg-blue-500 text-white rounded-lg font-medium hover:bg-blue-600 active:bg-blue-700 transition-colors"
            >
              Ir a perfil
            </button>
            <button
              @click="addToWhitelist(user)"
              class="min-h-[44px] px-4 py-2 border border-gray-300 rounded-lg font-medium hover:bg-gray-50 active:bg-gray-100 transition-colors"
              title="Añadir a whitelist"
            >
              + WL
            </button>
          </div>
        </div>
      </div>

      <!-- Desktop Table (visible on medium+ screens) -->
      <div class="hidden md:block bg-white border border-gray-200 rounded-xl overflow-hidden shadow-sm">
        <table class="w-full">
          <thead class="bg-gray-50 border-b border-gray-200">
            <tr>
              <th class="px-6 py-4 text-left text-sm font-semibold text-gray-700">Username</th>
              <th class="px-6 py-4 text-left text-sm font-semibold text-gray-700">Fecha seguido</th>
              <th class="px-6 py-4 text-right text-sm font-semibold text-gray-700">Acciones</th>
            </tr>
          </thead>
          <tbody class="divide-y divide-gray-100">
            <tr
              v-for="user in filteredResults"
              :key="user.href"
              class="hover:bg-gray-50 transition-colors"
            >
              <td class="px-6 py-4">
                <span class="font-medium text-gray-800">@{{ user.username }}</span>
              </td>
              <td class="px-6 py-4 text-gray-600">{{ formatDate(user.timestamp) }}</td>
              <td class="px-6 py-4 text-right">
                <div class="flex justify-end gap-2">
                  <button
                    @click="openProfile(user.href)"
                    class="px-4 py-2 bg-blue-500 text-white rounded-lg text-sm font-medium hover:bg-blue-600 transition-colors"
                  >
                    Ir a perfil
                  </button>
                  <button
                    @click="addToWhitelist(user)"
                    class="px-4 py-2 border border-gray-300 rounded-lg text-sm font-medium hover:bg-gray-50 transition-colors"
                  >
                    + Whitelist
                  </button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Empty state when filtered -->
      <div v-if="filteredResults.length === 0 && results.length > 0" class="text-center py-8 text-gray-500">
        <p class="text-4xl mb-2">🔍</p>
        <p>No se encontraron resultados para "{{ searchQuery }}"</p>
      </div>
    </section>

    <!-- Empty state when no results at all -->
    <div v-else-if="results.length === 0" class="text-center py-12 text-gray-500">
      <p class="text-4xl mb-2">✨</p>
      <p class="text-lg font-medium">¡Todos te siguen de vuelta!</p>
      <p class="text-sm mt-1">O aún no has cargado los archivos</p>
    </div>
  `
};
```

**Step 2: Add handler for whitelist in main app**

```javascript
const addToWhitelist = (username) => {
  whitelistSet.value.add(username.toLowerCase());
  saveWhitelist();
  processData(); // Re-process to remove from results
  showToast(`@${username} añadido a whitelist`);
};
```

**Step 3: Add to template**

```html
<ResultsSection
  :results="results"
  :whitelist-set="whitelistSet"
  @add-to-whitelist="addToWhitelist"
/>
```

**Step 4: Test search and sort**

1. Load sample data
2. Search for "not" - should filter results
3. Change sort order - verify order changes

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add ResultsSection with mobile cards and desktop table"
```

---

## Task 6: Toast Notifications

**Files:**
- Modify: `index.html`

**Step 1: Add Toast component**

```javascript
const Toast = {
  props: {
    message: String,
    visible: Boolean
  },
  emits: ['undo', 'close'],
  template: `
    <Transition
      enter-active-class="transition-all duration-300 ease-out"
      enter-from-class="translate-y-full opacity-0"
      enter-to-class="translate-y-0 opacity-100"
      leave-active-class="transition-all duration-200 ease-in"
      leave-from-class="translate-y-0 opacity-100"
      leave-to-class="translate-y-full opacity-0"
    >
      <div
        v-if="visible"
        class="fixed bottom-4 left-4 right-4 md:left-auto md:right-4 md:w-auto md:min-w-[300px] bg-gray-800 text-white px-4 py-3 rounded-lg shadow-lg flex items-center justify-between gap-4 z-50"
      >
        <span>✅ {{ message }}</span>
        <div class="flex gap-2">
          <button
            @click="$emit('undo')"
            class="text-blue-300 hover:text-blue-200 font-medium"
          >
            Deshacer
          </button>
          <button
            @click="$emit('close')"
            class="text-gray-400 hover:text-gray-300"
          >
            ✕
          </button>
        </div>
      </div>
    </Transition>
  `
};
```

**Step 2: Add toast state and methods to main app**

```javascript
const toast = reactive({
  visible: false,
  message: '',
  lastAction: null
});

let toastTimeout = null;

const showToast = (message, undoAction = null) => {
  if (toastTimeout) clearTimeout(toastTimeout);

  toast.message = message;
  toast.lastAction = undoAction;
  toast.visible = true;

  toastTimeout = setTimeout(() => {
    toast.visible = false;
  }, 5000);
};

const hideToast = () => {
  toast.visible = false;
  if (toastTimeout) clearTimeout(toastTimeout);
};

const undoLastAction = () => {
  if (toast.lastAction) {
    toast.lastAction();
    toast.visible = false;
  }
};

// Update addToWhitelist to support undo
const addToWhitelist = (username) => {
  const lowerUsername = username.toLowerCase();
  whitelistSet.value.add(lowerUsername);
  saveWhitelist();
  processData();

  showToast(`@${username} añadido a whitelist`, () => {
    whitelistSet.value.delete(lowerUsername);
    saveWhitelist();
    processData();
  });
};
```

**Step 3: Add to template**

```html
<Toast
  :message="toast.message"
  :visible="toast.visible"
  @undo="undoLastAction"
  @close="hideToast"
/>
```

**Step 4: Test undo functionality**

1. Add user to whitelist
2. Click "Deshacer" within 5 seconds
3. Verify user reappears in results

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add Toast notifications with undo support"
```

---

## Task 7: Whitelist Manager Component

**Files:**
- Modify: `index.html`

**Step 1: Add WhitelistManager component**

```javascript
const WhitelistManager = {
  props: {
    whitelistSet: { type: Set, default: () => new Set() }
  },
  emits: ['add', 'remove', 'export'],
  setup(props, { emit }) {
    const isExpanded = ref(false);
    const newUsername = ref('');

    const whitelistArray = computed(() => {
      return Array.from(props.whitelistSet).sort();
    });

    const addUsername = () => {
      const username = newUsername.value.trim().toLowerCase().replace('@', '');
      if (username && !props.whitelistSet.has(username)) {
        emit('add', username);
        newUsername.value = '';
      }
    };

    const removeUsername = (username) => {
      emit('remove', username);
    };

    const exportWhitelist = () => {
      emit('export');
    };

    return {
      isExpanded,
      newUsername,
      whitelistArray,
      addUsername,
      removeUsername,
      exportWhitelist
    };
  },
  template: `
    <div class="mb-6">
      <button
        @click="isExpanded = !isExpanded"
        class="w-full flex items-center justify-between p-4 bg-white border border-gray-200 rounded-lg hover:bg-gray-50 transition-colors"
      >
        <span class="flex items-center gap-2 font-medium text-gray-700">
          <span>📋</span>
          <span>Whitelist ({{ whitelistSet.size }})</span>
        </span>
        <span class="text-gray-400">{{ isExpanded ? '▼' : '▶' }}</span>
      </button>

      <div v-show="isExpanded" class="mt-3 p-4 bg-white border border-gray-200 rounded-lg">
        <!-- Add username input -->
        <div class="flex gap-2 mb-4">
          <input
            v-model="newUsername"
            @keyup.enter="addUsername"
            type="text"
            placeholder="@username"
            class="flex-1 px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none"
          />
          <button
            @click="addUsername"
            class="px-4 py-2 bg-blue-500 text-white rounded-lg font-medium hover:bg-blue-600 transition-colors min-h-[44px]"
          >
            Añadir
          </button>
        </div>

        <!-- Whitelist items -->
        <div v-if="whitelistArray.length > 0" class="space-y-2 max-h-60 overflow-y-auto mb-4">
          <div
            v-for="username in whitelistArray"
            :key="username"
            class="flex items-center justify-between p-2 bg-gray-50 rounded-lg"
          >
            <span class="text-gray-700">@{{ username }}</span>
            <button
              @click="removeUsername(username)"
              class="text-red-500 hover:text-red-700 p-1"
              title="Eliminar de whitelist"
            >
              🗑️
            </button>
          </div>
        </div>

        <div v-else class="text-center py-4 text-gray-500">
          <p>Whitelist vacía</p>
          <p class="text-sm">Añade cuentas que quieras ignorar</p>
        </div>

        <!-- Export button -->
        <button
          v-if="whitelistArray.length > 0"
          @click="exportWhitelist"
          class="w-full px-4 py-2 border border-gray-300 rounded-lg font-medium hover:bg-gray-50 transition-colors"
        >
          📥 Exportar whitelist.json
        </button>
      </div>
    </div>
  `
};
```

**Step 2: Add whitelist management methods to main app**

```javascript
const removeFromWhitelist = (username) => {
  whitelistSet.value.delete(username.toLowerCase());
  saveWhitelist();
  processData();
  showToast(`@${username} eliminado de whitelist`);
};

const exportWhitelist = () => {
  const data = JSON.stringify(Array.from(whitelistSet.value), null, 2);
  const blob = new Blob([data], { type: 'application/json' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = 'whitelist.json';
  a.click();
  URL.revokeObjectURL(url);
  showToast('Whitelist exportada');
};

const addToWhitelistManual = (username) => {
  whitelistSet.value.add(username.toLowerCase());
  saveWhitelist();
  processData();
  showToast(`@${username} añadido a whitelist`);
};
```

**Step 3: Add to template**

```html
<WhitelistManager
  :whitelist-set="whitelistSet"
  @add="addToWhitelistManual"
  @remove="removeFromWhitelist"
  @export="exportWhitelist"
/>
```

**Step 4: Test whitelist management**

1. Add username manually
2. Remove username
3. Export and verify JSON file downloads

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add WhitelistManager component with export"
```

---

## Task 8: Final Assembly and Polish

**Files:**
- Modify: `index.html`
- Create: `README.md`

**Step 1: Assemble complete index.html**

Ensure all components are registered and template is complete.

**Step 2: Create README.md**

```markdown
# Instagram Follower Analyzer

Herramienta para visualizar qué cuentas de Instagram no te siguen de vuelta.

## Uso

1. Abre `index.html` en tu navegador
2. Descarga tus datos de Instagram:
   - Instagram → Configuración → Tu actividad → Descargar información
   - Selecciona formato JSON
3. Carga los archivos:
   - `followers_*.json` en "Followers"
   - `following.json` en "Following"
   - (Opcional) Tu `whitelist.json`
4. Revisa los resultados y gestiona tu whitelist

## Características

- 📱 Mobile-first responsive design
- 🔍 Búsqueda y filtrado de resultados
- 📋 Whitelist persistente (localStorage)
- ⚠️ Detección de edge cases (duplicados, usernames cambiados, datos antiguos)
- 📥 Exportar whitelist como JSON

## Privacidad

- Todo el procesamiento ocurre localmente en tu navegador
- Ningún dato se envía a servidores externos
- Los archivos JSON nunca salen de tu dispositivo

## Estructura de archivos de Instagram

### followers.json (Formato A)
\`\`\`json
[{ "title": "username", "string_list_data": [{ "href": "...", "value": "...", "timestamp": ... }] }]
\`\`\`

### following.json (Formato B)
\`\`\`json
{ "relationships_following": [{ "title": "username", "string_list_data": [...] }] }
\`\`\`
```

**Step 3: Final testing**

1. Test on mobile viewport (Chrome DevTools)
2. Test all warning types
3. Test localStorage persistence (reload page)
4. Test whitelist export/import cycle

**Step 4: Final commit**

```bash
git add index.html README.md
git commit -m "feat: complete Instagram Follower Analyzer v1.0"
```

---

## Summary

| Task | Description | Est. Steps |
|------|-------------|------------|
| 1 | Project scaffold | 6 |
| 2 | DropZone component | 3 |
| 3 | JSON Parser & Normalizer | 4 |
| 4 | Warnings Panel | 4 |
| 5 | Results Table | 5 |
| 6 | Toast Notifications | 5 |
| 7 | Whitelist Manager | 5 |
| 8 | Final Assembly | 4 |

**Total: 8 tasks, ~36 steps**
