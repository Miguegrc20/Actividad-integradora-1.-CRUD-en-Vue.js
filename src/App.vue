<script setup>
import { computed, onMounted, reactive, ref } from 'vue'

const storageKey = 'biblioteca-crud-libros'
const searchTerm = ref('')
const selectedStatus = ref('Todos')
const showForm = ref(false)
const editingId = ref(null)
const formError = ref('')
const books = ref([])

const emptyForm = () => ({
  title: '',
  author: '',
  year: '',
  genre: 'Novela',
  status: 'Pendiente',
  synopsis: '',
})

const form = reactive(emptyForm())

const filteredBooks = computed(() => {
  const query = searchTerm.value.trim().toLowerCase()

  return books.value.filter((book) => {
    const matchesSearch = !query || [book.title, book.author, book.genre]
      .some((value) => value.toLowerCase().includes(query))
    const matchesStatus = selectedStatus.value === 'Todos' || book.status === selectedStatus.value
    return matchesSearch && matchesStatus
  })
})

const readingCount = computed(() => books.value.filter((book) => book.status === 'Leyendo').length)
const finishedCount = computed(() => books.value.filter((book) => book.status === 'Terminado').length)

function saveBooks() {
  localStorage.setItem(storageKey, JSON.stringify(books.value))
}

function resetForm() {
  Object.assign(form, emptyForm())
  editingId.value = null
  formError.value = ''
}

function openCreateForm() {
  resetForm()
  showForm.value = true
}

function editBook(book) {
  Object.assign(form, book)
  editingId.value = book.id
  formError.value = ''
  showForm.value = true
  window.scrollTo({ top: 0, behavior: 'smooth' })
}

function closeForm() {
  showForm.value = false
  resetForm()
}

function submitForm() {
  formError.value = ''

  if (!form.title.trim() || !form.author.trim() || !form.year || !form.synopsis.trim()) {
    formError.value = 'Completa los campos obligatorios para guardar el libro.'
    return
  }

  const bookData = {
    title: form.title.trim(),
    author: form.author.trim(),
    year: Number(form.year),
    genre: form.genre,
    status: form.status,
    synopsis: form.synopsis.trim(),
  }

  if (editingId.value) {
    const index = books.value.findIndex((book) => book.id === editingId.value)
    books.value[index] = { ...books.value[index], ...bookData }
  } else {
    books.value.unshift({ id: crypto.randomUUID(), ...bookData })
  }

  saveBooks()
  closeForm()
}

function deleteBook(bookId) {
  const book = books.value.find((item) => item.id === bookId)
  if (!book || !window.confirm(`¿Eliminar “${book.title}” de tu biblioteca?`)) return

  books.value = books.value.filter((item) => item.id !== bookId)
  saveBooks()
}

function statusClass(status) {
  return status.toLowerCase().replace('é', 'e')
}

onMounted(() => {
  const storedBooks = localStorage.getItem(storageKey)
  books.value = storedBooks ? JSON.parse(storedBooks) : [
    {
      id: crypto.randomUUID(),
      title: 'Cien años de soledad',
      author: 'Gabriel García Márquez',
      year: 1967,
      genre: 'Realismo mágico',
      status: 'Terminado',
      synopsis: 'La historia de la familia Buendía y del pueblo de Macondo a través de siete generaciones.',
    },
    {
      id: crypto.randomUUID(),
      title: 'El problema de los tres cuerpos',
      author: 'Cixin Liu',
      year: 2008,
      genre: 'Ciencia ficción',
      status: 'Leyendo',
      synopsis: 'Una señal extraterrestre conecta el destino de la humanidad con una civilización al borde del colapso.',
    },
    {
      id: crypto.randomUUID(),
      title: 'Sapiens',
      author: 'Yuval Noah Harari',
      year: 2011,
      genre: 'Historia',
      status: 'Pendiente',
      synopsis: 'Un recorrido por la evolución de la humanidad y las ideas que han transformado nuestro mundo.',
    },
  ]

  if (!storedBooks) saveBooks()
})
</script>

<template>
  <main class="app-shell">
    <header class="topbar">
      <a class="brand" href="#inicio" aria-label="Biblioteca, inicio">
        <span class="brand-mark">B</span>
        <span>Biblioteca</span>
      </a>
      <span class="topbar-note">Colección personal <span class="topbar-dot"></span> {{ books.length }} títulos</span>
    </header>

    <section id="inicio" class="hero-section">
      <div class="hero-copy">
        <p class="eyebrow">MI ESTANTERÍA DIGITAL</p>
        <h1>Libros que<br /><em>dejan huella.</em></h1>
        <p class="hero-intro">Organiza tus lecturas, descubre tus próximos mundos y lleva el pulso de tu colección.</p>
        <button class="primary-button" type="button" @click="openCreateForm">
          <span class="button-plus">+</span> Agregar libro
        </button>
      </div>
      <div class="hero-art" aria-hidden="true">
        <div class="sun-disc"></div>
        <div class="book-stack stack-back"></div>
        <div class="book-stack stack-mid"></div>
        <div class="book-stack stack-front"><span>LECTURAS<br />2026</span></div>
        <div class="art-caption">UNA PÁGINA<br />A LA VEZ</div>
      </div>
    </section>

    <section class="library-section" aria-labelledby="library-heading">
      <div class="section-heading">
        <div>
          <p class="eyebrow">TU COLECCIÓN</p>
          <h2 id="library-heading">Todos los libros</h2>
        </div>
        <div class="stats" aria-label="Resumen de lecturas">
          <span><strong>{{ books.length }}</strong> total</span>
          <span><strong>{{ readingCount }}</strong> leyendo</span>
          <span><strong>{{ finishedCount }}</strong> terminados</span>
        </div>
      </div>

      <div class="toolbar">
        <label class="search-box">
          <span class="search-icon">⌕</span>
          <span class="sr-only">Buscar libros</span>
          <input v-model="searchTerm" type="search" placeholder="Buscar por título, autor o género..." />
        </label>
        <div class="filter-group" aria-label="Filtrar por estado">
          <button v-for="status in ['Todos', 'Leyendo', 'Pendiente', 'Terminado']" :key="status" type="button" :class="{ active: selectedStatus === status }" @click="selectedStatus = status">
            {{ status }}
          </button>
        </div>
      </div>

      <div v-if="filteredBooks.length" class="book-grid">
        <article v-for="book in filteredBooks" :key="book.id" class="book-card">
          <div class="card-topline">
            <span class="book-index">{{ String(books.indexOf(book) + 1).padStart(2, '0') }}</span>
            <span class="status-badge" :class="statusClass(book.status)">{{ book.status }}</span>
          </div>
          <div class="book-cover" :class="`cover-${book.genre.toLowerCase().replaceAll(' ', '-')}`">
            <span class="cover-label">{{ book.genre }}</span>
            <span class="cover-title">{{ book.title }}</span>
            <span class="cover-author">{{ book.author }}</span>
          </div>
          <div class="book-details">
            <div class="book-heading">
              <h3>{{ book.title }}</h3>
              <span class="book-year">{{ book.year }}</span>
            </div>
            <p class="author">{{ book.author }}</p>
            <p class="synopsis">{{ book.synopsis }}</p>
            <div class="card-actions">
              <button type="button" class="text-button" @click="editBook(book)">Editar <span>↗</span></button>
              <button type="button" class="delete-button" aria-label="Eliminar libro" @click="deleteBook(book.id)">Eliminar</button>
            </div>
          </div>
        </article>
      </div>
      <div v-else class="empty-state">
        <span class="empty-icon">⌕</span>
        <h3>No encontramos libros</h3>
        <p>Prueba con otra búsqueda o agrega un nuevo título a tu colección.</p>
        <button type="button" class="text-button" @click="searchTerm = ''; selectedStatus = 'Todos'">Limpiar filtros</button>
      </div>
    </section>

    <footer class="footer">
      <span>Biblioteca personal</span>
      <span>CRUD de libros · Vue.js</span>
    </footer>

    <div v-if="showForm" class="modal-backdrop" @click.self="closeForm">
      <section class="modal" role="dialog" aria-modal="true" aria-labelledby="form-title">
        <button type="button" class="close-button" aria-label="Cerrar formulario" @click="closeForm">×</button>
        <p class="eyebrow">{{ editingId ? 'EDITAR REGISTRO' : 'NUEVO REGISTRO' }}</p>
        <h2 id="form-title">{{ editingId ? 'Actualiza tu libro' : 'Agrega un libro' }}</h2>
        <p class="modal-intro">Los campos marcados con * son obligatorios.</p>
        <form @submit.prevent="submitForm">
          <div class="form-grid">
            <label class="field field-wide">Título *<input v-model="form.title" type="text" placeholder="Ej. Pedro Páramo" /></label>
            <label class="field">Autor *<input v-model="form.author" type="text" placeholder="Nombre del autor" /></label>
            <label class="field">Año *<input v-model="form.year" type="number" min="1" max="2100" placeholder="2026" /></label>
            <label class="field">Género<select v-model="form.genre"><option>Novela</option><option>Realismo mágico</option><option>Ciencia ficción</option><option>Historia</option><option>Ensayo</option><option>Poesía</option><option>Otro</option></select></label>
            <label class="field">Estado<select v-model="form.status"><option>Pendiente</option><option>Leyendo</option><option>Terminado</option></select></label>
            <label class="field field-wide">Sinopsis *<textarea v-model="form.synopsis" rows="4" placeholder="Escribe una breve descripción..."></textarea></label>
          </div>
          <p v-if="formError" class="form-error" role="alert">{{ formError }}</p>
          <div class="form-actions">
            <button type="button" class="secondary-button" @click="closeForm">Cancelar</button>
            <button type="submit" class="primary-button">{{ editingId ? 'Guardar cambios' : 'Guardar libro' }}</button>
          </div>
        </form>
      </section>
    </div>
  </main>
</template>
