<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Agenda de Eventos de Dança</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f4f4f4;
            margin: 0;
            padding: 20px;
        }
        .calendar {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
            gap: 10px;
            margin-top: 20px;
        }
        .day {
            background: white;
            padding: 20px;
            border: 1px solid #ddd;
            text-align: center;
            border-radius: 5px;
            position: relative;
            cursor: pointer;
            transition: background 0.3s;
        }
        .day.red {
            background: #ffcccb; /* Vermelho claro */
        }
        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .month-title {
            font-size: 24px;
        }
        .button {
            padding: 5px 10px;
            cursor: pointer;
        }
        .modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.7);
            justify-content: center;
            align-items: center;
        }
        .modal-content {
            background: white;
            padding: 20px;
            border-radius: 5px;
            width: 90%; /* Ajustado para dispositivos móveis */
            max-width: 400px;
            position: relative;
        }
        .close {
            position: absolute;
            top: 10px;
            right: 10px;
            cursor: pointer;
            color: red;
        }
        .event-details {
            margin-top: 10px;
            text-align: left;
        }
        .add-event-button {
            position: fixed;
            bottom: 20px;
            right: 20px;
            background: #007bff;
            color: white;
            border: none;
            border-radius: 50%;
            width: 50px;
            height: 50px;
            font-size: 24px;
            cursor: pointer;
        }
        textarea {
            width: 100%;
            height: 60px;
            margin-top: 10px;
            resize: none;
        }
        .share-link {
            margin-top: 20px;
        }
    </style>
</head>
<body>

    <h1>Agenda de Eventos de Dança</h1>

    <div class="header">
        <button class="button" onclick="prevMonth()">Anterior</button>
        <div class="month-title" id="month-title"></div>
        <button class="button" onclick="nextMonth()">Próximo</button>
    </div>

    <div class="calendar" id="calendar"></div>

    <div class="modal" id="modal">
        <div class="modal-content">
            <span class="close" onclick="closeModal()">&times;</span>
            <h3 id="selected-day"></h3>
            <div class="event-details" id="event-details"></div>
        </div>
    </div>

    <button class="add-event-button" onclick="openAddEventModal()">+</button>

    <div class="modal" id="add-event-modal">
        <div class="modal-content">
            <span class="close" onclick="closeAddEventModal()">&times;</span>
            <h3>Adicionar Evento</h3>
            <p id="event-date"></p>
            <form id="event-form">
                <input type="text" id="event-name" placeholder="Nome do Evento" required>
                <input type="text" id="event-location" placeholder="Localização" required>
                <input type="time" id="event-time" required>
                <input type="text" id="event-contact" placeholder="Contato" required>
                <input type="file" id="event-photo" accept="image/*">
                <textarea id="event-observation" placeholder="Observação (opcional)"></textarea>
                <input type="hidden" id="event-day" required>
                <button type="submit">Adicionar Evento</button>
            </form>
        </div>
    </div>

    <div class="share-link">
        <strong>Compartilhe o link:</strong>
        <input type="text" id="share-url" readonly>
        <button onclick="copyLink()">Copiar</button>
    </div>

    <script>
        const monthNames = ["Janeiro", "Fevereiro", "Março", "Abril", "Maio", "Junho", "Julho", "Agosto", "Setembro", "Outubro", "Novembro", "Dezembro"];
        let currentMonth = new Date().getMonth();
        let currentYear = new Date().getFullYear();
        const events = {};

        function renderCalendar() {
            const calendar = document.getElementById('calendar');
            const monthTitle = document.getElementById('month-title');
            calendar.innerHTML = '';
            monthTitle.textContent = `${monthNames[currentMonth]} ${currentYear}`;

            const firstDay = new Date(currentYear, currentMonth, 1).getDay();
            const daysInMonth = new Date(currentYear, currentMonth + 1, 0).getDate();

            for (let i = 0; i < firstDay; i++) {
                calendar.innerHTML += `<div class="day"></div>`;
            }

            for (let day = 1; day <= daysInMonth; day++) {
                const eventKey = `${currentYear}-${currentMonth + 1}-${day}`;
                calendar.innerHTML += `
                    <div class="day" id="day-${eventKey}" onclick="showEventDetails('${eventKey}', ${day})" onmouseover="setEventDay(${day})">
                        ${day}
                        <div id="event-${eventKey}"></div>
                    </div>`;
            }
            updateShareLink();
        }

        function prevMonth() {
            if (currentMonth === 0) {
                currentMonth = 11;
                currentYear--;
            } else {
                currentMonth--;
            }
            renderCalendar();
        }

        function nextMonth() {
            if (currentMonth === 11) {
                currentMonth = 0;
                currentYear++;
            } else {
                currentMonth++;
            }
            renderCalendar();
        }

        function setEventDay(day) {
            document.getElementById('event-day').value = day;
            document.getElementById('event-date').textContent = `Adicionar Evento no dia ${day}/${currentMonth + 1}/${currentYear}`;
        }

        function openAddEventModal() {
            document.getElementById('event-form').reset();
            document.getElementById('add-event-modal').style.display = 'flex';
        }

        function closeAddEventModal() {
            document.getElementById('add-event-modal').style.display = 'none';
        }

        function showEventDetails(eventKey, day) {
            const eventList = events[eventKey] || [];
            document.getElementById('selected-day').textContent = `Eventos para ${day}/${currentMonth + 1}/${currentYear}`;
            document.getElementById('event-details').innerHTML = eventList.map(event => `
                <div>
                    <strong>${event.name}</strong><br>
                    Localização: ${event.location}<br>
                    Horário: ${event.time}<br>
                    Contato: ${event.contact}<br>
                    ${event.photo ? `<img src="${event.photo}" style="width:100%; max-width:200px;"/>` : ''}
                    <p>Observação: ${event.observation}</p>
                </div>
                <hr>
            `).join('');
            document.getElementById('modal').style.display = 'flex';
        }

        function closeModal() {
            document.getElementById('modal').style.display = 'none';
        }

        document.getElementById('event-form').onsubmit = function(e) {
            e.preventDefault();
            const name = document.getElementById('event-name').value;
            const location = document.getElementById('event-location').value;
            const time = document.getElementById('event-time').value;
            const contact = document.getElementById('event-contact').value;
            const photoInput = document.getElementById('event-photo');
            const photoURL = photoInput.files[0] ? URL.createObjectURL(photoInput.files[0]) : '';
            const observation = document.getElementById('event-observation').value;
            const day = document.getElementById('event-day').value;

            const eventKey = `${currentYear}-${currentMonth + 1}-${day}`;
            if (!events[eventKey]) {
                events[eventKey] = [];
            }
            events[eventKey].push({ name, location, time, contact, photo: photoURL, observation });

            const dayElement = document.getElementById(`day-${eventKey}`);
            dayElement.classList.add('red'); // Muda para vermelho se houver evento

            closeAddEventModal();
            renderCalendar();
        };

        function copyLink() {
            const linkInput = document.getElementById('share-url');
            linkInput.select();
            document.execCommand('copy');
            alert('Link copiado para a área de transferência!');
        }

        function updateShareLink() {
            const linkInput = document.getElementById('share-url');
            linkInput.value = window.location.href;
        }

        renderCalendar();
    </script>

</body>
</html>
