# C-Users-zenaz-Documents-Codex-2026-06-09-files-mentioned-by-the-user-texto-outputs-index.html
      document.getElementById("btnCopyMessage").addEventListener("click", () => this.copyMessage());
      ["necesidad", "dolor", "participacion", "situacion_laboral", "ciclo", "modalidad", "canal_preferido"].forEach((id) => {
        document.getElementById(id).addEventListener("change", () => this.updatePreview());
      });
    },
    populateSelect(id, items, keepFirst = false) {
      const select = document.getElementById(id);
      const first = keepFirst ? select.innerHTML : "";
      select.innerHTML = first + items.map((item) => `<option>${item}</option>`).join("");
    },
    getFormStudent() {
      const ids = ["id_estudiante", "nombre", "correo", "telefono", "carrera", "ciclo", "modalidad", "situacion_laboral", "satisfaccion", "necesidad", "dolor", "inquietud", "canal_preferido", "participacion", "comentario"];
      return ids.reduce((data, id) => {
        data[id] = document.getElementById(id).value.trim();
        return data;
      }, {});
    },
    saveForm(event) {
      event.preventDefault();
      const form = event.currentTarget;
      if (!form.checkValidity()) {
        form.classList.add("was-validated");
        this.toast("Revise los campos obligatorios.");
        return;
      }
      const student = this.getFormStudent();
      const index = document.getElementById("editingIndex").value;
      const duplicate = Storage.getStudents().some((item, itemIndex) => item.correo.toLowerCase() === student.correo.toLowerCase() && String(itemIndex) !== String(index));
      if (duplicate) {
        this.toast("Ya existe un estudiante con ese correo.");
        return;
      }
      Storage.upsert(student, index);
      this.toast(index === "" ? "Estudiante registrado." : "Estudiante actualizado.");
      this.clearForm();
      this.refresh();
    },
    clearForm() {
      document.getElementById("studentForm").reset();
      document.getElementById("studentForm").classList.remove("was-validated");
      document.getElementById("editingIndex").value = "";
      document.getElementById("satisfaccion").value = 7;
      document.getElementById("satisfactionValue").textContent = "7";
      this.updatePreview();
    },
    updatePreview() {
      document.getElementById("satisfactionValue").textContent = document.getElementById("satisfaccion").value;
      const student = Business.enrichStudent(this.getFormStudent());
      document.getElementById("recommendationPreview").innerHTML = `
        <div><span class="segment-badge">${student.segmento}</span> <span class="risk-badge ${this.riskClass(student.riesgo)}">Riesgo ${student.riesgo}</span></div>
        <div><strong>Recomendación CRM:</strong> ${student.crm.recomendacion} · ${student.crm.accion}</div>
        <div><strong>Canal:</strong> ${student.crm.canal} · <strong>Prioridad:</strong> ${student.crm.prioridad} · <strong>Seguimiento:</strong> ${student.crm.seguimiento}</div>
      `;
    },
    seedDemo() {
      const students = DemoData.create();
      Storage.saveStudents(students);
      this.toast(`Datos restaurados: ${students.length} registros.`);
      this.refresh();
    },
    refresh() {
      this.renderTable();
      this.renderMessageOptions();
      this.updatePreview();
      if (window.StudentDashboard) window.StudentDashboard.render(Storage.getStudents());
    },
    filteredStudents() {
      const query = document.getElementById("searchInput").value.toLowerCase();
      const risk = document.getElementById("riskFilter").value;
      const segment = document.getElementById("segmentFilter").value;
      return Storage.getStudents().filter((student) => {
        const text = `${student.nombre} ${student.correo} ${student.carrera}`.toLowerCase();
        return (!query || text.includes(query)) && (!risk || student.riesgo === risk) && (!segment || student.segmento === segment);
      });
    },
    renderTable() {
      const students = this.filteredStudents();
      const all = Storage.getStudents();
      document.getElementById("studentsTable").innerHTML = students.map((student) => {
        const index = all.findIndex((item) => item.id_estudiante === student.id_estudiante && item.correo === student.correo);
        return `
          <tr class="priority-${student.crm.prioridad.toLowerCase()}">
            <td><div class="student-name">${student.nombre}</div><div class="student-email">${student.correo}</div></td>
            <td>${student.carrera}<div class="student-email">Ciclo ${student.ciclo} · ${student.modalidad}</div></td>
            <td><strong>${student.satisfaccion}/10</strong><div class="student-email">${student.participacion}</div></td>
            <td><span class="segment-badge">${student.segmento}</span></td>
            <td><span class="risk-badge ${this.riskClass(student.riesgo)}">${student.riesgo}</span></td>
            <td>${student.necesidad}<div class="student-email">${student.dolor}</div></td>
            <td>${student.crm.recomendacion}<div class="student-email">${student.crm.seguimiento}</div></td>
            <td class="text-end">
              <button class="btn btn-sm btn-outline-primary" onclick="StudentCRM.UI.editStudent(${index})">Editar</button>
              <button class="btn btn-sm btn-outline-danger" onclick="StudentCRM.UI.deleteStudent(${index})">Eliminar</button>
            </td>
          </tr>
        `;
      }).join("") || `<tr><td colspan="8" class="text-center text-muted py-4">No hay estudiantes para mostrar.</td></tr>`;
    },
    editStudent(index) {
      const student = Storage.getStudents()[index];
      Object.keys(student).forEach((key) => {
        const field = document.getElementById(key);
        if (field) field.value = student[key];
      });
      document.getElementById("editingIndex").value = index;
      document.getElementById("satisfactionValue").textContent = student.satisfaccion;
      this.updatePreview();
      document.getElementById("registro").scrollIntoView({ behavior: "smooth" });
    },
    deleteStudent(index) {
      if (!confirm("¿Eliminar este estudiante?")) return;
      Storage.remove(index);
      this.toast("Estudiante eliminado.");
      this.refresh();
    },
    renderMessageOptions() {
      const students = Storage.getStudents();
      document.getElementById("messageStudent").innerHTML = students.map((student, index) => `<option value="${index}">${student.nombre} · ${student.segmento}</option>`).join("");
      this.renderMessage();
    },
    renderMessage() {
      const students = Storage.getStudents();
      const index = document.getElementById("messageStudent").value || 0;
      const channel = document.getElementById("messageChannel").value;
      const student = students[index];
      document.getElementById("messageOutput").value = student ? Business.buildMessage(student, channel) : "";
    },
    copyMessage() {
      const output = document.getElementById("messageOutput");
      output.select();
      navigator.clipboard.writeText(output.value).then(() => this.toast("Mensaje copiado."));
    },
    riskClass(risk) {
      return risk === "Alto" ? "risk-high" : risk === "Medio" ? "risk-medium" : "risk-low";
    },
    toast(message) {
      const toast = document.createElement("div");
      toast.className = "toast-lite";
      toast.textContent = message;
      document.body.appendChild(toast);
      setTimeout(() => toast.remove(), 2600);
    }
  };

  window.StudentCRM = { Storage, Business, Catalogs, DemoData, EventData, UI };
  document.addEventListener("DOMContentLoaded", () => UI.init());
})();
