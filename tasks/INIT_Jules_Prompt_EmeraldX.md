# **PROJECT: EmeraldX (React Port)**

**Role:** Senior Frontend Architect

**Objective:** Port the "EmeraldX Evidence Board" (originally Vanilla JS) to a robust, production-grade **React** application.

**Constraint:** Client-side only. No external backend.

## **1\. Tech Stack**

* **Core:** React 19, Vite, TypeScript.  
* **Styling:** Tailwind CSS (Utility-first).  
* **State Management:** zustand (Crucial for performance with high-frequency drag updates).  
* **Icons:** lucide-react.  
* **Persistence:** idb (Lightweight wrapper for IndexedDB).  
* **Utils:** uuid, clsx, tailwind-merge.

## **2\. Core Architecture**

The app is a "Z-Index Stack" rendered on an infinite canvas.

1. **UI Layer (z-1000):** Fixed position toolbar and panels.  
2. **Node Layer (z-100):** HTML Divs (React Components) for evidence content.  
3. **Connection Layer (z-50):** A single SVG element covering the entire coordinate space for drawing edges.  
4. **Group Layer (z-10):** Container nodes that visually group other nodes.

## **3\. Data Models (TypeScript Interfaces)**

### **Node**

interface Node {  
  id: string;  
  type: 'image' | 'note' | 'group';  
  subtype?: 'gallery' | 'cabinet' | 'drive'; // Only for groups  
  position: { x: number; y: number };  
  dimensions: { w: number; h: number };  
  content: {  
    title?: string;  
    data?: string; // Text content or Base64 Image string  
  };  
  meta: {  
    locked: boolean;  
    groupId?: string | null; // ID of parent group  
  };  
}

### **Edge**

interface Edge {  
  id: string;  
  source: string; // Node ID  
  target: string; // Node ID  
  controlPoint?: { x: number; y: number }; // Optional custom quadratic bezier point  
  style: {  
    color: string;  
    width: number;  
    opacity: number;  
    type: 'solid' | 'dashed' | 'arrow';  
    label?: string;  
  };  
}

### **BoardState (Zustand Store)**

interface BoardState {  
  view: { x: number; y: number; scale: number };  
  nodes: Node\[\];  
  edges: Edge\[\];  
  interactionMode: 'idle' | 'panning' | 'connecting' | 'link-mode';  
  // Actions...  
}

## **4\. Functional Requirements**

### **A. The Canvas (Viewport)**

* **Infinite Pan/Zoom:** Implement using CSS Transforms (translate3d \+ scale) on a CanvasWrapper component.  
* **Inputs:** Spacebar+Drag to pan. Wheel to zoom (clamp 0.25x to 5x).  
* **Counter-Scaling:** Anchors and Edge Handles must scale inversely (1 / scale) so they remain clickable at low zoom levels.

### **B. Node Logic**

* **Drag:** Moving a node updates its XY coordinates.  
* **Grouping Physics:** If a node is type group, dragging it must calculate the delta and apply it to all child nodes (nodes with meta.groupId \=== this.id).  
* **Auto-File:** Dropping a node onto a Group node visually updates its groupId.

### **C. Edge Logic (Smart Connections)**

* **Rendering:** Use SVG \<path\> with Bezier curves.  
* **Modes:**  
  * *Default:* Cubic Bezier (Auto-calculated S-curve).  
  * *Pinned:* Quadratic Bezier (If controlPoint exists).  
* **Link Mode:** A rapid-fire mode where clicking Source \-\> Target(s) creates edges instantly without dragging.

### **D. Persistence (IndexedDB)**

* **Why:** LocalStorage (5MB) is too small for images.  
* **Implementation:** Use idb. On app mount, load state. On any state change (debounced 1s), write full state to IndexedDB.

## **5\. Execution Plan (Phased)**

**Phase 1: Skeleton & State**

* Initialize Vite \+ TS \+ Tailwind.  
* Setup useBoardStore with Zustand.  
* Create the basic Canvas component that handles Pan/Zoom transforms.

**Phase 2: Nodes & Interaction**

* Implement NodeComponent.  
* Implement Drag-and-Drop logic (recommend use-gesture or native event listeners for performance).  
* Implement Group logic (parent/child movement).

**Phase 3: The Edge Layer**

* Implement ConnectionLayer (SVG).  
* Add the logic for drawing lines between anchors.  
* Implement the "Edge Editor" popup (Context menu).

**Phase 4: Persistence & Polish**

* Integrate idb for saving/loading.  
* Add the "Link Mode" toolbar interactions.  
* Add Image Upload (Base64 conversion).

**Start with Phase 1\.**