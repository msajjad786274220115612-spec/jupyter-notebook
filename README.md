// === LocalStorage Safe Wrapper ===
const getData = (key) => JSON.parse(localStorage.getItem(key) || "[]");
const setData = (key, value) => localStorage.setItem(key, JSON.stringify(value));

// === Notebooks Array ===
let notebooks = getData("notebooks");

// === Create Notebook ===
const createNotebook = (title = "نیا نوٹ پیڈ", color = "#fff") => {
  const nb = { id: Date.now(), title, color, createdAt: new Date(), notes: [] };
  notebooks = notebooks.concat(nb); // functional
  setData("notebooks", notebooks);
  renderNotebooks();
};

// === Delete Notebook ===
const deleteNotebook = (notebookId) => {
  notebooks = notebooks.filter(nb => nb.id !== notebookId);
  setData("notebooks", notebooks);
  renderNotebooks();
};

// === Add Note ===
const addNote = (notebookId, text) => {
  if(!text.trim()) return;
  notebooks = notebooks.map(nb => nb.id === notebookId 
    ? { ...nb, notes: nb.notes.concat({ id: Date.now(), text, timestamp: new Date() }) } 
    : nb);
  setData("notebooks", notebooks);
  renderNotebooks();
};

// === Edit Note ===
const editNote = (notebookId, noteId, newText) => {
  notebooks = notebooks.map(nb => nb.id === notebookId 
    ? { ...nb, notes: nb.notes.map(n => n.id === noteId ? { ...n, text: newText } : n) } 
    : nb);
  setData("notebooks", notebooks);
};

// === Delete Note ===
const deleteNote = (notebookId, noteId) => {
  notebooks = notebooks.map(nb => nb.id === notebookId 
    ? { ...nb, notes: nb.notes.filter(n => n.id !== noteId) } 
    : nb);
  setData("notebooks", notebooks);
  renderNotebooks();
};

// === Sort Notes ===
const sortNotes = (notebookId, order = "newest") => {
  notebooks = notebooks.map(nb => nb.id === notebookId
    ? { ...nb, notes: nb.notes.slice().sort((a,b)=> order==="newest" ? b.timestamp - a.timestamp : a.timestamp - b.timestamp) }
    : nb);
  renderNotebooks();
};

// === Render Notebooks ===
const renderNotebooks = () => {
  const container = document.getElementById("notebooksContainer");
  container.innerHTML = "";
  notebooks.forEach(nb => {
    const nbDiv = document.createElement("div");
    nbDiv.className = "notebook";
    nbDiv.style.background = nb.color;
    nbDiv.innerHTML = `
      <div class="notebook-header">
        <h3>${nb.title}</h3>
        <button onclick="deleteNotebook(${nb.id})">🗑️ حذف کریں</button>
      </div>
      <input placeholder="نیا نوٹ لکھیں" id="noteInput-${nb.id}">
      <button onclick="addNoteUI(${nb.id})">نوٹ شامل کریں</button>
      <div>
        Sort by:
        <select onchange="sortNotes(${nb.id}, this.value)">
          <option value="newest">تازہ ترین</option>
          <option value="oldest">پرانا</option>
        </select>
      </div>
      <div id="notes-${nb.id}"></div>
    `;
    container.appendChild(nbDiv);
    renderNotesUI(nb.id);
  });
};

// === Render Notes ===
const renderNotesUI = (notebookId) => {
  const nb = notebooks.find(nb=>nb.id===notebookId);
  const notesContainer = document.getElementById(`notes-${notebookId}`);
  notesContainer.innerHTML = "";
  nb.notes.forEach(n => {
    const nDiv = document.createElement("div");
    nDiv.className = "note";
    nDiv.innerHTML = `
      <textarea onchange="editNote(${notebookId}, ${n.id}, this.value)">${n.text}</textarea>
      <button onclick="deleteNote(${notebookId}, ${n.id})">حذف کریں</button>
      <small>${new Date(n.timestamp).toLocaleString()}</small>
    `;
    notesContainer.appendChild(nDiv);
  });
};

// === UI Helpers ===
const addNoteUI = (notebookId) => {
  const input = document.getElementById(`noteInput-${notebookId}`);
  addNote(notebookId, input.value);
  input.value = "";
};

// Initial render
renderNotebooks();
