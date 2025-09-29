<template>
  <div class="calendar-container">
    <div class="calendar-header">
      <h1 class="calendar-title">Event Calendar</h1>
      <div class="calendar-actions">
        <button @click="addEvent" class="btn-primary">
          <i class="icon-plus"></i> Add Event
        </button>
        <button @click="toggleView" class="btn-secondary">
          {{ currentView === 'dayGridMonth' ? 'Events View' : 'Month View' }}
        </button>
      </div>
    </div>
    
    <div class="calendar-wrapper">
      <FullCalendar
        v-if="currentView !== 'eventsList'"
        ref="fullCalendar"
        :options="calendarOptions"
        class="modern-calendar"
      />
      <div v-else class="events-list-view">
        <div class="events-list-header">
          <h3>All Events</h3>
          <span class="events-count">{{ events.length }} totals</span>
        </div>
        <div v-if="sortedEvents.length === 0" class="empty-events">No events yet. Click "Add Event" to create one.</div>
        <ul v-else class="events-list">
          <li v-for="event in sortedEvents" :key="event.id" class="event-item">
            <span class="event-color" :style="{ backgroundColor: event.backgroundColor }"></span>
            <div class="event-main">
              <div class="event-title">{{ event.title }}</div>
              <div class="event-datetime">{{ new Date(event.start).toLocaleString() }}<span v-if="event.end"> - {{ new Date(event.end).toLocaleString() }}</span></div>
            </div>
            <button class="btn-secondary event-edit" @click="handleEventClick({ event: { id: event.id, title: event.title, start: new Date(event.start), end: event.end ? new Date(event.end) : new Date(event.start), backgroundColor: event.backgroundColor } })">Edit</button>
          </li>
        </ul>
      </div>
    </div>

    <!-- Event Modal -->
    <div v-if="showModal" class="modal-overlay" @click="closeModal">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <h3>{{ isEditing ? 'Edit Event' : 'Add New Event' }}</h3>
          <button @click="closeModal" class="modal-close">&times;</button>
        </div>
        <form @submit.prevent="saveEvent" class="event-form">
          <div class="form-group">
            <label>Event Title</label>
            <input
              v-model="eventForm.title"
              type="text"
              required
              class="form-control"
              placeholder="Enter event title"
            />
          </div>
          <div class="form-group">
            <label>Start Date</label>
            <input
              v-model="eventForm.start"
              type="datetime-local"
              required
              class="form-control"
            />
          </div>
          <div class="form-group">
            <label>End Date</label>
            <input
              v-model="eventForm.end"
              type="datetime-local"
              required
              class="form-control"
            />
          </div>
          <div class="form-group">
            <label>Color</label>
            <select v-model="eventForm.backgroundColor" class="form-control">
              <option value="#3788d8">Blue</option>
              <option value="#28a745">Green</option>
              <option value="#ffc107">Yellow</option>
              <option value="#dc3545">Red</option>
              <option value="#6f42c1">Purple</option>
            </select>
          </div>
          <div class="form-actions">
            <button v-if="isEditing" type="button" @click="deleteEvent" class="btn-danger">
              Delete
            </button>
            <button type="submit" class="btn-primary">
              {{ isEditing ? 'Update' : 'Create' }} Event
            </button>
            <button type="button" @click="closeModal" class="btn-secondary">
              Cancel
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import FullCalendar from '@fullcalendar/vue3'
import dayGridPlugin from '@fullcalendar/daygrid'
import timeGridPlugin from '@fullcalendar/timegrid'
import interactionPlugin from '@fullcalendar/interaction'

export default {
  name: 'CalendarComponent',
  components: {
    FullCalendar
  },
  computed: {
    sortedEvents() {
      return [...this.events].sort((a, b) => new Date(a.start) - new Date(b.start))
    }
  },
  data() {
    return {
      currentView: 'dayGridMonth',
      showModal: false,
      isEditing: false,
      eventForm: {
        title: '',
        start: '',
        end: '',
        backgroundColor: '#3788d8'
      },
      events: [
        {
          id: '1',
          title: 'Team Meeting',
          start: new Date(),
          backgroundColor: '#3788d8',
          borderColor: '#2c5aa0'
        },
        {
          id: '2',
          title: 'Project Deadline',
          start: new Date(Date.now() + 86400000),
          backgroundColor: '#dc3545',
          borderColor: '#c82333'
        }
      ],
      calendarOptions: {
        plugins: [dayGridPlugin, timeGridPlugin, interactionPlugin],
        initialView: 'dayGridMonth',
        headerToolbar: {
          left: 'prev,next today',
          center: 'title',
          right: 'dayGridMonth,timeGridWeek,timeGridDay'
        },
        events: [],
        editable: true,
        selectable: true,
        selectMirror: true,
        dayMaxEvents: true,
        weekends: true,
        height: 'auto',
        aspectRatio: 1.35,
        eventClick: this.handleEventClick,
        select: this.handleDateSelect,
        eventDrop: this.handleEventDrop,
        eventResize: this.handleEventResize,
        themeSystem: 'standard'
      }
    }
  },
  mounted() {
    this.calendarOptions.events = this.events
  },
  methods: {
    addEvent() {
      this.isEditing = false
      this.resetForm()
      this.showModal = true
    },
    handleDateSelect(selectInfo) {
      this.isEditing = false
      this.eventForm.start = this.formatDateForInput(selectInfo.start)
      this.eventForm.end = this.formatDateForInput(selectInfo.end)
      this.showModal = true
    },
    handleEventClick(clickInfo) {
      const event = clickInfo.event
      this.isEditing = true
      this.eventForm = {
        id: event.id,
        title: event.title,
        start: this.formatDateForInput(event.start),
        end: this.formatDateForInput(event.end || event.start),
        backgroundColor: event.backgroundColor || '#3788d8'
      }
      this.showModal = true
    },
    saveEvent() {
      const newEvent = {
        id: this.isEditing ? this.eventForm.id : Date.now().toString(),
        title: this.eventForm.title,
        start: new Date(this.eventForm.start),
        end: new Date(this.eventForm.end),
        backgroundColor: this.eventForm.backgroundColor,
        borderColor: this.getBorderColor(this.eventForm.backgroundColor)
      }

      if (this.isEditing) {
        const index = this.events.findIndex(e => e.id === this.eventForm.id)
        if (index !== -1) {
          this.events.splice(index, 1, newEvent)
        }
      } else {
        this.events.push(newEvent)
      }

      this.calendarOptions.events = [...this.events]
      this.closeModal()
    },
    deleteEvent() {
      if (!this.isEditing || !this.eventForm.id) return
      const shouldDelete = confirm('Are you sure you want to delete this event?')
      if (!shouldDelete) return
      this.events = this.events.filter(e => e.id !== this.eventForm.id)
      this.calendarOptions.events = [...this.events]
      this.closeModal()
    },
    handleEventDrop(eventDropInfo) {
      const event = eventDropInfo.event
      const eventIndex = this.events.findIndex(e => e.id === event.id)
      if (eventIndex !== -1) {
        this.events[eventIndex] = {
          ...this.events[eventIndex],
          start: event.start,
          end: event.end
        }
      }
    },
    handleEventResize(eventResizeInfo) {
      const event = eventResizeInfo.event
      const eventIndex = this.events.findIndex(e => e.id === event.id)
      if (eventIndex !== -1) {
        this.events[eventIndex] = {
          ...this.events[eventIndex],
          start: event.start,
          end: event.end
        }
      }
    },
    toggleView() {
      this.currentView = this.currentView === 'dayGridMonth' ? 'eventsList' : 'dayGridMonth'
      if (this.currentView !== 'eventsList') {
        this.calendarOptions.initialView = this.currentView
      }
    },
    closeModal() {
      this.showModal = false
      this.resetForm()
    },
    resetForm() {
      this.eventForm = {
        title: '',
        start: '',
        end: '',
        backgroundColor: '#3788d8'
      }
    },
    formatDateForInput(date) {
      if (!date) return ''
      const d = new Date(date)
      return d.toISOString().slice(0, 16)
    },
    getBorderColor(backgroundColor) {
      const colorMap = {
        '#3788d8': '#2c5aa0',
        '#28a745': '#1e7e34',
        '#ffc107': '#d39e00',
        '#dc3545': '#c82333',
        '#6f42c1': '#5a32a3'
      }
      return colorMap[backgroundColor] || '#2c5aa0'
    },
    async loadEvents() {
      try {
        const response = await fetch('/api/events')
        const events = await response.json()
        this.events = events
        this.calendarOptions.events = events
      } catch (error) {
        console.error('Error loading events:', error)
      }
    },
    async saveEventToAPI(event) {
      try {
        const response = await fetch('/api/events', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify(event)
        })
        return await response.json()
      } catch (error) {
        console.error('Error saving event:', error)
      }
    }
    }
}
</script>

<style scoped>
.calendar-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  min-height: 100vh;
}

.calendar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
  padding: 20px;
  background: white;
  border-radius: 15px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
}

.calendar-title {
  font-size: 2.5rem;
  color: #2c3e50;
  margin: 0;
  font-weight: 700;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  background-clip: text;
  -webkit-background-clip: text;
  color: transparent;
  -webkit-text-fill-color: transparent;
}

.calendar-actions {
  display: flex;
  gap: 15px;
}

.btn-primary, .btn-secondary, .btn-danger {
  padding: 12px 24px;
  border: none;
  border-radius: 25px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 8px;
}

.btn-primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(102, 126, 234, 0.4);
}

.btn-secondary {
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
  color: white;
}

.btn-secondary:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(245, 87, 108, 0.4);
}

.btn-danger {
  background: linear-gradient(135deg, #ff6b6b 0%, #c53030 100%);
  color: white;
}

.btn-danger:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(197, 48, 48, 0.4);
}

.calendar-wrapper {
  background: white;
  border-radius: 20px;
  padding: 30px;
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

/* Events List View */
.events-list-view {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.events-list-header {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  margin-bottom: 8px;
}

.events-list-header h3 {
  margin: 0;
  font-size: 1.25rem;
  color: #2d3748;
}

.events-count {
  color: #718096;
  font-weight: 600;
}

.empty-events {
  color: #718096;
  background: #f7fafc;
  border: 1px dashed #e2e8f0;
  padding: 16px;
  border-radius: 12px;
  text-align: center;
}

.events-list {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.event-item {
  display: grid;
  grid-template-columns: 10px 1fr auto;
  gap: 12px;
  align-items: center;
  padding: 12px 16px;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  background: linear-gradient(135deg, #ffffff 0%, #f9fafb 100%);
}

.event-color {
  width: 10px;
  height: 10px;
  border-radius: 2px;
}

.event-main {
  display: flex;
  flex-direction: column;
}

.event-title {
  font-weight: 700;
  color: #2d3748;
}

.event-datetime {
  color: #4a5568;
  font-size: 0.9rem;
}

.event-edit {
  white-space: nowrap;
}

/* FullCalendar Custom Styles */
:deep(.fc) {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
}

:deep(.fc-header-toolbar) {
  margin-bottom: 30px;
  padding: 15px 0;
}

:deep(.fc-button-primary) {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border: none;
  border-radius: 10px;
  padding: 8px 16px;
  font-weight: 600;
}

:deep(.fc-button-primary:hover) {
  background: linear-gradient(135deg, #5a67d8 0%, #6b46c1 100%);
  transform: translateY(-1px);
}

:deep(.fc-daygrid-day) {
  border: 1px solid #e2e8f0;
}

:deep(.fc-daygrid-day:hover) {
  background-color: #f8fafc;
}

:deep(.fc-event) {
  border-radius: 8px;
  border: none;
  padding: 2px 6px;
  font-weight: 500;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
  transition: all 0.2s ease;
}

:deep(.fc-event:hover) {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

:deep(.fc-title) {
  font-size: 1.5rem;
  font-weight: 700;
  color: #2d3748;
}

:deep(.fc-col-header-cell) {
  background: linear-gradient(135deg, #f7fafc 0%, #edf2f7 100%);
  font-weight: 600;
  color: #4a5568;
  padding: 15px 0;
}

/* Modal Styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  backdrop-filter: blur(5px);
}

.modal-content {
  background: white;
  border-radius: 20px;
  width: 90%;
  max-width: 500px;
  max-height: 80vh;
  overflow-y: auto;
  box-shadow: 0 25px 50px rgba(0, 0, 0, 0.25);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 25px 30px 0;
  border-bottom: 1px solid #e2e8f0;
  margin-bottom: 25px;
}

.modal-header h3 {
  margin: 0;
  font-size: 1.5rem;
  color: #2d3748;
  font-weight: 700;
}

.modal-close {
  background: none;
  border: none;
  font-size: 2rem;
  color: #a0aec0;
  cursor: pointer;
  padding: 0;
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  transition: all 0.2s ease;
}

.modal-close:hover {
  background: #f7fafc;
  color: #4a5568;
}

.event-form {
  padding: 0 30px 30px;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #4a5568;
}

.form-control {
  width: 100%;
  padding: 12px 16px;
  border: 2px solid #e2e8f0;
  border-radius: 10px;
  font-size: 1rem;
  transition: border-color 0.2s ease;
  box-sizing: border-box;
}

.form-control:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.form-actions {
  display: flex;
  gap: 15px;
  justify-content: flex-end;
  margin-top: 30px;
}

/* Responsive Design */
@media (max-width: 768px) {
  .calendar-container {
    padding: 10px;
  }
  
  .calendar-header {
    flex-direction: column;
    gap: 15px;
  }
  
  .calendar-title {
    font-size: 2rem;
  }
  
  .calendar-wrapper {
    padding: 15px;
  }
  
  .modal-content {
    width: 95%;
    margin: 20px;
  }
  
  :deep(.fc-header-toolbar) {
    flex-direction: column;
    gap: 10px;
  }
  
  :deep(.fc-toolbar-chunk) {
    display: flex;
    justify-content: center;
  }
}

/* ADDITIONAL CUSTOM CSS*/

/* Add to CalendarComponent.vue styles */
:deep(.fc-event-main) {
  font-weight: 500;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
}

:deep(.fc-daygrid-day-number) {
  font-weight: 600;
  color: #4a5568;
}

:deep(.fc-day-today) {
  background: linear-gradient(135deg, #e6fffa 0%, #b2f5ea 100%) !important;
}

:deep(.fc-daygrid-day-events) {
  margin-top: 4px;
}

/* Hover effects for days */
:deep(.fc-daygrid-day:hover) {
  background: linear-gradient(135deg, #f7fafc 0%, #edf2f7 100%);
  cursor: pointer;
}

</style>
