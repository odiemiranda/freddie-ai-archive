---
source: notebooklm
notebook: 559134a2
topic: "Tool Runner Research"
question: "Looking at the auto-discovery router pattern: should the tool name map 1:1 to the directory/file name, or should there be an alias system? For example, our notebooklm tool is at src/tools/notebooklm/index.ts — should the user type 'bun run tool notebooklm' or could they also type 'bun run tool research' (an alias)? What is the simplest way to support aliases without a registry file? Also: should the router support '--list' to show all available tools, and '--help' to show a tool's help?"
date: 2026-04-03
---

# Looking at the auto-discovery router pattern: should the tool name map 1:1 to the directory/file name, or should there be an alias system? For example, our notebooklm tool is at src/tools/notebooklm/index.ts — should the user type 'bun run tool notebooklm' or could they also type 'bun run tool research' (an alias)? What is the simplest way to support aliases without a registry file? Also: should the router support '--list' to show all available tools, and '--help' to show a tool's help?

For a project with 20+ tools, moving to intent-based aliases (like `research`) rather than implementation-based names (like `notebooklm`) is a great idea. It aligns with good CLI design principles where commands reflect user goals rather than internal architecture.

Here is how you should handle aliases, `--list`, and `--help` while keeping the router as simple as possible.

### 1. The Simplest Way to Support Aliases (Without a Registry File)
You have two great options that avoid creating a complex, separate registry file:

**Option A: File System Symlinks (The "Zero-Code" Approach)**
Since you are using a convention-based router that checks the file system, you can natively support aliases by creating symbolic links in your `src/tools/` directory. 
If you run `ln -s notebooklm/index.ts research.ts` in your tools folder, your existing router will automatically discover `research` without a single line of code changed. 

**Option B: An In-Router Dictionary (The "Pragmatic" Approach)**
If you dislike symlinks (e.g., for cross-platform Git reasons), the simplest alternative is a tiny, hardcoded dictionary directly at the top of your `router.ts` file. It avoids a separate registry file while keeping your routing logic explicitly visible.
```typescript
const aliases: Record<string, string> = {
  research: "notebooklm",
  db: "database-admin"
};
```

### 2. Should the router support `--list`?
**Yes.** Even as a single developer, once you pass 20 tools, you will inevitably forget the exact names of tools you wrote months ago. 

You can easily implement this by using `readdir` from `node:fs/promises` to scan the `src/tools/` directory [1, 2] and printing the discovered files and folders.

### 3. Should the router support `--help`?
**Delegate it.** Because you are using `Bun.spawn` to isolate your tools, the router should *not* try to parse or display the tool's help text. 

*   **If the user types `bun run tool --help` (no tool name):** The router should intercept this, show generic usage instructions, and automatically display the `--list` of available tools [3].
*   **If the user types `bun run tool research --help`:** The router should simply discover the `research` tool and pass the `--help` flag directly to it as a standard argument [4, 5]. The tool itself (using a library like Commander or meow) will intercept the flag and print its specific help menu [4, 5].

### The Updated Router Implementation

Here is how your `src/router.ts` looks when combining these concepts:

```typescript
import { readdir } from "node:fs/promises";

// 1. Pragmatic In-Router Aliases
const aliases: Record<string, string> = {
  research: "notebooklm",
};

const args = Bun.argv.slice(2);
const command = args;
const toolArgs = args.slice(1);

// 2. Intercept --help and --list at the Router level
if (!command || command === "--help" || command === "-h" || command === "--list") {
  console.log("Usage: bun run tool <name|alias> [args]\n");
  console.log("Available Tools:");
  
  // Dynamically scan the tools directory
  const entries = await readdir("src/tools", { withFileTypes: true });
  for (const entry of entries) {
    if (entry.isDirectory() || entry.name.endsWith(".ts")) {
      const toolName = entry.name.replace(".ts", "");
      console.log(`  - ${toolName}`);
    }
  }
  
  console.log("\nAliases:");
  for (const [alias, target] of Object.entries(aliases)) {
    console.log(`  - ${alias} -> ${target}`);
  }
  process.exit(0);
}

// 3. Resolve Alias -> Tool Name
const resolvedName = aliases[command] || command;

// 4. Auto-Discovery
const standardPath = `src/tools/${resolvedName}.ts`;
const indexPath = `src/tools/${resolvedName}/index.ts`;
let targetPath = "";

if (await Bun.file(standardPath).exists()) {
  targetPath = standardPath;
} else if (await Bun.file(indexPath).exists()) {
  targetPath = indexPath;
} else {
  console.error(`❌ Tool or alias '${command}' not found.`);
  process.exit(1);
}

// 5. Delegate Execution (Passes --help or other args down to the tool)
const proc = Bun.spawn(["bun", targetPath, ...toolArgs], {
  stdio: ["inherit", "inherit", "inherit"],
});

await proc.exited;
process.exit(proc.exitCode);
```

## References

1. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
2. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
3. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
4. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
5. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
6. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
7. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
8. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
9. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
10. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
11. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
12. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
13. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
14. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
15. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
16. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
17. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
18. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
19. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
20. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
21. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
22. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
23. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
24. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
25. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
26. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
27. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
28. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
29. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
30. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
31. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
32. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
33. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
34. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
35. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
36. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
37. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
38. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
39. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
40. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
41. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
42. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
43. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
44. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
45. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
46. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
47. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
48. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
49. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
50. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
51. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
52. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
53. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
54. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
55. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
56. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
57. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
58. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
59. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
60. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
61. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
62. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
63. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
64. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
65. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
66. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
67. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
68. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
69. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
70. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
71. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
72. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
73. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
74. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
75. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
76. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
77. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
78. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
79. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
80. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
81. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
82. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
83. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
84. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
85. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
86. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
87. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
88. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
89. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
90. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
91. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
92. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
93. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
94. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
95. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
96. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
97. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
98. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
99. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
100. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
101. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
102. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
103. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
104. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
105. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
106. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
107. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
108. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
109. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
110. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
111. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
112. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
113. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
114. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
115. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
116. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
117. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
118. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
119. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
120. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
121. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
122. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
123. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
124. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
125. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
126. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
127. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
128. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
129. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
130. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
131. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
132. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
133. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
134. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
135. [93d9dd9d-5bd2-448b-9cff-eef736d18489] — "We have a index.ts file and that’s where we will keep all the code for our CLI program. Open the index.ts file and replace its content with the following import statements."
136. [5ff380d6-f226-40c3-95f0-13abe13e33b5]
137. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "The help information is auto-generated based on the information commander already knows about your program. The default help option is  -h,--help ."
138. [07974382-d601-4e1b-9c01-fbbe6bb514f5] — "For information about terms used in this document see: terminology"
139. [4e04d30f-1188-47b4-92df-ffe347d8619c] — "Here we used the  meow(helpText, options)  function to define the program’s behaviour. We gave it the help text to display and defined the flags in the second parameter. The  flags  object specifies the flags as keys along with their type and optional  shortFlag  (or alias)."
