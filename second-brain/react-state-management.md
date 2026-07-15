There are two state  
Server State: Async data owner by remote server  
Client State: Synchronous data owned entirely by browser  

React Query  
Before this, we got server state using useEffect and clent state management (Extensive). It reduces useEffect and useState usage by 90%  

Patterns need to be followed  

Eg. A list drawer. On clicking on one item it opens a detailed view on right. On the right panel one can edit as well.
The list is still open while view/edit of the list item.   
Question: If we use react queries and it can re fetch things if (React Query refetch conditions), then how does the screen not crash and remain stable.  
Solutons:  
(Its called a Master/Detail dashboard challenge)
Problem: If the edit form is directly wired to read the active item prop, a background update of the query will whatever user had typed.
For this we can use "Decoupled ID" Architecture
Idea is to remove structural navigation (item selected) from active editing(data inside form inputs). Instead of passing the entire object we got from react query we pass only the id.
We retrieve the list via react query and render list, and have a state as selectedItemId, all list are wrapped in a button, onclick trigger setSelectedItemId. So we attached the id to a state.
We pass this set id to The Detail/Edit panel. This components triggers getDetails(id) via react query and we pass that to Edit Compnent/ Details component. for detaisl part we need no state managemnet, we can directly use react query one.
But for the Edit Coponent we need decoupling. So we send te object but copy data to a state inside it and based on that state and not prop we render the prefilled form.
Now if initialData changes, yes it causes reredner as prop changed. But it does not cause state reset, so text inside form is stillt there.
This Decoupled ID Architecture prevents background cache re-fetches from wiping out unsubmitted user inputs. While any background data update to itemDetails will trigger a standard React re-render by passing a new initialData object down to the <EditForm>, it will not cause a state reset. This works because the useState hook inside <EditForm> only evaluates its initial state argument once when the component initially mounts. On all subsequent re-renders forced by prop changes, React preserves the existing local state slot in memory and completely ignores the updated initialData prop. Consequently, the user's active keystrokes stay completely safe, isolated, and stable.

```javascript
import { useState } from 'react';
import { useQuery } from '@tanstack/react-query';

// ==========================================
// 1. THE PARENT DASHBOARD
// ==========================================
export function Dashboard() {
  // Track ONLY the ID of the active item in client state
  const [selectedId, setSelectedId] = useState<string | null>(null);

  // Fetch the list (React Query handles automatic updates safely)
  const { data: items = [] } = useQuery({
    queryKey: ['itemsList'],
    queryFn: fetchItemsList,
  });

  return (
    <div style={{ display: 'flex' }}>
      {/* LEFT: The Drawer List */}
      <div className="drawer">
        {items.map((item) => (
          <button key={item.id} onClick={() => setSelectedId(item.id)}>
            {item.name}
          </button>
        ))}
      </div>

      {/* RIGHT: Detail & Edit Panel */}
      <div className="right-panel">
        {selectedId && <DetailAndEditPanel itemId={selectedId} />}
      </div>
    </div>
  );
}

// ==========================================
// 2. THE DETAIL & EDIT CONTAINER
// ==========================================
function DetailAndEditPanel({ itemId }: { itemId: string }) {
  const [isEditing, setIsEditing] = useState(false);

  // Fetch the highly specific details for just THIS single item
  const { data: itemDetails, isLoading } = useQuery({
    queryKey: ['itemDetails', itemId],
    queryFn: () => fetchItemDetailsById(itemId),
    // Crucial: Prevent tab-switching background fetches from firing mid-edit
    staleTime: 1000 * 60 * 5, 
    refetchOnWindowFocus: false, 
  });

  if (isLoading || !itemDetails) return <div>Loading details...</div>;

  if (isEditing) {
    // When switching to edit mode, we mount the form and pass initial data
    return (
      <EditForm 
        initialData={itemDetails} 
        onCancel={() => setIsEditing(false)} 
      />
    );
  }

  return (
    <div>
      <h2>{itemDetails.name}</h2>
      <p>{itemDetails.description}</p>
      <button onClick={() => setIsEditing(true)}>Edit Item</button>
    </div>
  );
}

// ==========================================
// 3. THE ISOLATED EDIT FORM
// ==========================================
function EditForm({ initialData, onCancel }) {
  // Sever the connection to the server state entirely using local state
  const [formFields, setFormFields] = useState({
    name: initialData.name,
    description: initialData.description,
  });

  // If the background list changes now, it cannot touch 'formFields'
  return (
    <form onSubmit={(e) => e.preventDefault()}>
      <input
        value={formFields.name}
        onChange={(e) => setFormFields({ ...formFields, name: e.target.value })}
      />
      <textarea
        value={formFields.description}
        onChange={(e) => setFormFields({ ...formFields, description: e.target.value })}
      />
      <button type="submit">Save Changes</button>
      <button type="button" onClick={onCancel}>Cancel</button>
    </form>
  );
}

```

React Query Avoids Unnecessary Child Re-rendersReact Query has a built-in optimization feature called structural sharing.If a background re-fetch happens, but the server returns the exact same data as before, React Query keeps the exact same reference pointer in JavaScript memory. Because the reference does not change, React sees the prop as identical and will not unnecessarily re-render your child component   

React Query refetch conditions
1. Component Mount: Fires immediately when a component using useQuery first appears on the screen.  
2. Query Key Changes: Fires instantly when any dependency inside the queryKey array changes (like switching itemId).  
3. Window Focus Reset: Fires in the background whenever a user clicks away from your browser tab and clicks back into it.  
4. Network Reconnection: Fires automatically the moment a device regains a lost internet connection.  
5. Explicit Cache Invalidation: Fires manually when you call queryClient.invalidateQueries() after a successful mutation.

-------------------------
UI = function(state). For changing UI we change state and react patches only changed DOM

Rendering
A component re-reners only in 4 cases
1. It mounts
2. Its own setState / dispatch is called
3. Its parent re-rendered
4. A context it reads changed value

These are not triggers
1. Mutating normat variable or a state directly
2. Changing ref.current
3. props changing (which only happens if parent rerenders)

When a parent component passes a new prop down, it forces the child component to re-render. However, React completely protects existing state slots from being reset or modified during this re-render.  
State Updates ──► DO Cause a Re-render  
Prop Changes / Re-renders ──► Do NOT Change State  




