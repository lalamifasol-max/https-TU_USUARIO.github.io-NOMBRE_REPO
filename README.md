# https-TU_USUARIO.github.io-NOMBRE_REPO
<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <title>Chatbot - Itinerario Luna de Miel</title>
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <style>
    body { font-family: Arial, sans-serif; background:#f2f6fb; margin:0; padding:0; display:flex; justify-content:center; align-items:flex-start; min-height:100vh; }
    .container { width:100%; max-width:900px; margin:24px; background:#fff; border-radius:8px; box-shadow:0 6px 18px rgba(0,0,0,.08); overflow:hidden; }
    header { background:#0b63d6; color:#fff; padding:18px 20px; }
    header h1 { margin:0; font-size:18px; }
    .main { display:flex; gap:20px; padding:20px; }
    .kb { width:36%; max-height:66vh; overflow:auto; border-right:1px solid #eee; padding-right:12px; }
    .chatwrap { width:64%; display:flex; flex-direction:column; }
    .kb h2 { font-size:15px; margin:6px 0; }
    .kb pre { background:#f7f9fc; padding:10px; border-radius:6px; font-size:13px; white-space:pre-wrap; }
    .chat { flex:1; background:#eef6ff; padding:12px; border-radius:6px; overflow:auto; min-height:320px; max-height:66vh; }
    .msg { margin:8px 0; display:flex; gap:8px; }
    .user { justify-content:flex-end; }
    .bubble { padding:10px 12px; border-radius:12px; max-width:78%; font-size:14px; line-height:1.3; }
    .bot .bubble { background:#fff; color:#111; border:1px solid #e3eefc; }
    .user .bubble { background:#0b63d6; color:#fff; }
    .input { display:flex; gap:8px; margin-top:12px; }
    input[type="text"] { flex:1; padding:10px 12px; border-radius:8px; border:1px solid #d1e6ff; font-size:14px; }
    button { padding:10px 14px; border-radius:8px; border:0; background:#0b63d6; color:#fff; font-weight:600; cursor:pointer; }
    .small { font-size:13px; color:#666; margin-top:8px; }
    .tagbtn { display:inline-block; margin:6px 6px 0 0; padding:6px 8px; background:#eef6ff; border-radius:6px; cursor:pointer; font-size:13px; color:#0b63d6; }
  </style>
</head>
<body>
  <div class="container" role="main">
    <header>
      <h1>Asistente de Luna de Miel — Itinerario Maldives & Destinos recomendados</h1>
    </header>
    <div class="main">
      <div class="kb" aria-label="Base de conocimiento">
        <h2>Resumen rápido</h2>
        <pre id="kb-summary"></pre>

        <h2>Itinerario 7 días (Maldives)</h2>
        <pre id="kb-itinerary"></pre>

        <h2>Información faltante</h2>
        <pre id="kb-missing"></pre>

        <h2>Comandos útiles</h2>
        <div>
          <span class="tagbtn" data-q="Hola">Hola</span>
          <span class="tagbtn" data-q="¿Qué incluye el Día 3?">¿Qué incluye el Día 3?</span>
          <span class="tagbtn" data-q="¿Cómo llego a Malé?">¿Cómo llego a Malé?</span>
          <span class="tagbtn" data-q="Mostrar todo el itinerario">Mostrar todo el itinerario</span>
        </div>
      </div>

      <div class="chatwrap">
        <div class="chat" id="chat" aria-live="polite"></div>

        <div class="input">
          <input id="input" type="text" placeholder="Escribe tu pregunta (ej: ¿Qué hay en el Día 2?)" />
          <button id="send">Enviar</button>
        </div>
        <div class="small">Sugerencia: prueba preguntas como "¿Qué destinos recomiendan?" o "¿Hay buceo disponible?"</div>
      </div>
    </div>
  </div>

  <script>
    // Base de conocimiento: resumen, itinerario y datos faltantes
    const kb = {
      summary: "Documento: propuesta 'Innovación para el sector turismo' — foco en lunas de miel 2025. Recomendaciones: Maldives, Bali, Tanzania+Zanzibar, Costa Rica, Santorini. Incluye un itinerario de 7 días en Maldivas (actividades: snorkel, isla local, deportes acuáticos, spa, picnic en sandbank, avistamiento de delfines). Falta información operativa: contactos, precios, horarios y políticas.",
      itinerary: [
        "Día 1 — Llegada a Malé: Llegada al Aeropuerto Internacional de Malé (MLE); traslado al alojamiento; tiempo libre y puesta de sol.",
        "Día 2 — Snorkel y relax: Excursión de snorkel (posible avistamiento de tortugas y mantas); tarde libre para piscina o spa.",
        "Día 3 — Isla local y cultura: Visita a isla local; degustación de mas huni; compras de artesanías.",
        "Día 4 — Deportes acuáticos: Kayak, paddleboard o windsurf; opcional clase introductoria de buceo; cena en la playa.",
        "Día 5 — Delfines y picnic: Excursión para ver delfines (amanecer o atardecer) y picnic en un sandbank.",
        "Día 6 — Spa y actividades del resort: Día de spa, yoga, clases de cocina, fotografía subacuática; cena de despedida.",
        "Día 7 — Regreso: Mañana libre, check-out y traslado al Aeropuerto de Malé para el vuelo de regreso."
      ],
      missing: [
        "Nombres y direcciones de alojamientos y proveedores",
        "Teléfonos, emails y webs de contacto",
        "Precios (alojamiento, excursiones, transportes, comidas)",
        "Horarios de actividades y duración exacta",
        "Políticas de reserva/cancelación y seguros",
        "Información de accesibilidad y emergencias",
        "Requisitos de visado y salud"
      ]
    };

    // Preguntas y respuestas (extracto de las 45+ Q&A preparadas)
    const qas = [
      {q:"¿Qué es este documento?", a:"Es una propuesta de viaje para lunas de miel con recomendaciones de destinos y un itinerario de 7 días en Maldivas."},
      {q:"¿Qué destinos recomiendan para lunas de miel?", a:"Maldives, Bali, Tanzania + Zanzibar, Costa Rica y Santorini están recomendados."},
      {q:"¿Cuánto dura el itinerario en Maldivas?", a:"Siete días."},
      {q:"¿Dónde llego para el viaje a Maldivas?", a:"Al Aeropuerto Internacional de Malé (MLE)."},
      {q:"¿Incluye traslado desde el aeropuerto?", a:"El itinerario implica traslados desde Malé al alojamiento, pero no se indica si están incluidos ni el precio."},
      {q:"¿Qué hago en el Día 1?", a:"Llegada, traslado al resort/guesthouse y tiempo libre para la playa y la puesta de sol."},
      {q:"¿Qué incluye el Día 2?", a:"Excursión de snorkel y tarde libre para piscina o spa."},
      {q:"¿Qué pasa en el Día 3?", a:"Visita a una isla local, degustación de comida local y compras de artesanías."},
      {q:"¿Qué actividades hay el Día 4?", a:"Deportes acuáticos (kayak, paddleboard, windsurf) y una opción de clase introductoria de buceo; cena en la playa."},
      {q:"¿Cuándo es la excursión de delfines?", a:"Día 5, en horario de amanecer o atardecer — la hora exacta no está especificada."},
      {q:"¿Qué es un picnic en sandbank?", a:"Es un picnic privado en una pequeña barra de arena (sandbank) para disfrutar vistas y privacidad."},
      {q:"¿Qué incluye el Día 6?", a:"Spa, actividades del resort como yoga y clase de cocina, fotografía subacuática y cena de despedida."},
      {q:"¿Qué incluye el Día 7?", a:"Mañana libre, check-out y traslado al aeropuerto."},
      {q:"¿Puedo bucear?", a:"Sí, se menciona una clase introductoria de buceo como opción."},
      {q:"¿Hay snorkel?", a:"Sí, hay una excursión de snorkel programada en el itinerario."},
      {q:"¿Hay spa disponible?", a:"Sí, se mencionan tratamientos de spa en el itinerario."},
      {q:"¿Dónde compro souvenirs?", a:"En la visita a la isla local se mencionan artesanías y compras."},
      {q:"¿Están los precios en el documento?", a:"No, no hay precios ni tarifas incluidas."},
      {q:"¿Cómo reservo actividades?", a:"El documento no especifica el proceso de reserva; lo normal sería gestionarlo con el resort o proveedor local."},
      {q:"¿Hay teléfonos o contactos?", a:"No, no hay teléfonos, emails ni sitios web en el documento."},
      {q:"¿Hay información de seguridad o emergencias?", a:"No, esa información no está incluida."},
      {q:"¿Cuál es el mejor momento para ir?", a:"El documento no detalla la mejor temporada; no especifica estacionalidad."},
      {q:"¿Incluye comida?", a:"Se mencionan cenas en la playa y gastronomía local, pero no hay un plan de comidas detallado."},
      {q:"¿Hay Wi‑Fi?", a:"No se confirma; probablemente los resorts lo ofrecen pero no está especificado."},
      {q:"¿Se puede personalizar el itinerario?", a:"El documento sugiere opciones (por ejemplo, buceo opcional) pero no explica cómo personalizarlo."},
      {q:"¿Hay políticas de cancelación?", a:"No, no están incluidas en el documento."},
      {q:"¿Qué idiomas se hablan localmente?", a:"El documento no lo dice; el idioma oficial de Maldivas es el dhivehi (no listado en el documento)."},
      {q:"¿Se recomiendan seguros de viaje?", a:"El documento no lo especifica, pero se recomienda contratar seguro de viaje."},
      {q:"¿Se permiten niños?", a:"No se trata específicamente, el enfoque es en lunas de miel (parejas)."},
      {q:"¿Qué equipo llevar para snorkel?", a:"El documento no da lista; por lo general, traje de baño, protector solar 'reef-safe' y toalla. El equipo suele proveerlo el proveedor."},
      {q:"¿Cuánto dura la excursión de delfines?", a:"No se especifica la duración."},
      {q:"¿Es privado el picnic en sandbank?", a:"Se sugiere una experiencia privada, pero el nivel de privacidad no está confirmado."},
      {q:"¿Hay transporte entre islas?", a:"Se implican traslados en barco para excursiones e inter‑islas, pero no hay horarios ni proveedores."},
      {q:"¿Qué debo preguntar al resort al reservar?", a:"Pregunta por: traslados incluidos, horarios de actividades, precios, políticas de cancelación, servicios de accesibilidad y número de emergencia."},
      {q:"¿Cómo obtengo la conversión de PDF a Excel?", a:"Usa la herramienta 'PDF to Excel' en tu panel de Tools (no incluida en este chatbot)."},
      {q:"¿Cuál es el aeropuerto más cercano?", a:"Aeropuerto Internacional de Malé (MLE)."},
      {q:"¿Hay restaurantes en la isla local?", a:"Se menciona gastronomía local y restaurantes del resort, pero no nombres ni horarios."},
      {q:"¿Qué hago si necesito asistencia médica?", a:"El documento no incluye contactos de emergencia; solicita esa información al resort o a la autoridad local antes de viajar."},
      {q:"¿Qué idiomas soporta este bot?", a:"Este bot está en español, pero se puede adaptar a otros idiomas si lo despliegas con contenidos traducidos."}
    ];

    // JSON intents mapping (respuestas cortas)
    const intents = {
      greeting: "Hola — soy tu asistente de luna de miel. ¿En qué puedo ayudarte?",
      hours: "No hay horarios exactos en el documento; las actividades están indicadas por día.",
      address: "No hay direcciones concretas; sólo se menciona el Aeropuerto Internacional de Malé como punto de llegada.",
      directions: "Llegar a Malé (MLE). Los traslados locales en barco/lancha están implicados pero no detallados.",
      tickets: "No se especifican instrucciones de reserva ni compra de tickets en el documento.",
      prices: "No hay precios ni costos incluidos.",
      facilities: "Se mencionan resorts/guesthouses, spa, piscina, gastronomía local y actividades del resort; detalles no confirmados.",
      accessibility: "No hay información de accesibilidad en el documento.",
      tours: "Tours mencionados: snorkel, avistamiento de delfines, visita a isla local, clase de buceo introductoria y actividades del resort.",
      events: "Eventos del resort: yoga, clase de cocina, fotografía subacuática, spa y cena de despedida.",
      rules: "No hay reglas o normativas especificadas en el documento.",
      contact: "No hay teléfonos, emails ni sitios web en el documento.",
      openingSeason: "No se especifican temporadas; el documento no detalla la mejor época.",
      bestTimeToVisit: "No especificado en el documento.",
      nearestTransport: "Aeropuerto Internacional de Malé (MLE).",
      emergency: "No hay contactos de emergencia incluidos.",
      refundPolicy: "No hay política de reembolso/cancelación incluida.",
      recommendation: "Destinos recomendados: Maldives, Bali, Tanzania+Zanzibar, Costa Rica y Santorini.",
      fallback: "Lo siento, no entendí. Puedes preguntar sobre el itinerario de Maldivas, actividades o qué información falta en el documento."
    };

    // Poner contenido en la columna KB
    document.getElementById("kb-summary").textContent = kb.summary;
    document.getElementById("kb-itinerary").textContent = kb.itinerary.join("\\n");
    document.getElementById("kb-missing").textContent = kb.missing.join("\\n");

    // Chat rendering
    const chat = document.getElementById("chat");
    const input = document.getElementById("input");
    const send = document.getElementById("send");

    function addMessage(text, from='bot') {
      const wrapper = document.createElement('div');
      wrapper.className = 'msg ' + (from === 'user' ? 'user' : 'bot');
      const bubble = document.createElement('div');
      bubble.className = 'bubble';
      bubble.textContent = text;
      wrapper.appendChild(bubble);
      chat.appendChild(wrapper);
      chat.scrollTop = chat.scrollHeight;
    }

    function findExactAnswer(userText) {
      const lower = userText.trim().toLowerCase();
      for (const pair of qas) {
        if (pair.q.toLowerCase() === lower) return pair.a;
      }
      return null;
    }

    function findKeywordAnswer(userText) {
      const low = userText.toLowerCase();
      // keywords mapping
      const kwMap = [
        {k:['itinerario','7','siete','día','dia'], a: kb.itinerary.join("\\n")},
        {k:['destinos','recomi','recomend'], a: "Destinos recomendados: Maldives, Bali, Tanzania + Zanzibar, Costa Rica y Santorini."},
        {k:['malé','mle','aeropuerto'], a: "Aeropuerto Internacional de Malé (MLE) es el punto de llegada para Maldivas."},
        {k:['snorkel','snórkel','snor'], a: "Sí, hay una excursión de snorkel (Día 2) con posibilidad de ver tortugas y mantas."},
        {k:['delfin','delfines'], a: "Excursión para ver delfines está en el Día 5, normalmente al amanecer o atardecer; la hora exacta no se especifica."},
        {k:['spa','masaje'], a: "El itinerario incluye días de spa y tratamientos (Día 2 y Día 6), pero no hay instrucciones de reserva en el documento."},
        {k:['precio','cost','cuesta','tarifa'], a: intents.prices},
        {k:['contacto','telefono','teléfono','email','correo','web','website'], a: intents.contact},
        {k:['reserv','book','reserva','cancel'], a: "El documento no detalla procedimientos de reserva ni políticas de cancelación; consulta directamente con el resort o proveedor."},
        {k:['accesibilidad','accesible','wheelchair'], a: intents.accessibility},
        {k:['urgencia','emergencia','salud','vacuna'], a: intents.emergency},
        {k:['mostrar todo','mostrar itinerario','ver todo','ver itinerario'], a: kb.itinerary.join("\\n")}
      ];
      for (const map of kwMap) {
        for (const kk of map.k) if (low.includes(kk)) return map.a;
      }
      return null;
    }

    function handleInput(raw) {
      const text = raw.trim();
      if (!text) return;
      addMessage(text, 'user');
      // buscar respuesta exacta
      let resp = findExactAnswer(text);
      if (resp) { addMessage(resp); return; }
      // buscar por keywords
      resp = findKeywordAnswer(text);
      if (resp) { addMessage(resp); return; }
      // check intents map by simple rules
      if (/hola|buenos|buenas|hey/i.test(text)) { addMessage(intents.greeting); return; }
      if (/ayuda|help|qué puedo/i.test(text)) { addMessage("Puedo responder sobre: el itinerario de 7 días, actividades (snorkel, spa, yoga), traslados a Malé, y qué información falta en el documento. Prueba: '¿Qué hay en el Día 2?'"); return; }
      // fallback
      addMessage(intents.fallback);
    }

    send.addEventListener('click', () => { handleInput(input.value); input.value=''; input.focus(); });
    input.addEventListener('keydown', (e) => { if (e.key === 'Enter') { handleInput(input.value); input.value=''; } });

    // tag buttons
    document.querySelectorAll('.tagbtn').forEach(btn => {
      btn.addEventListener('click', () => {
        const q = btn.getAttribute('data-q');
        input.value = q;
        handleInput(q);
      });
    });

    // saludo inicial
    addMessage(intents.greeting);
  </script>
</body>
</html>
