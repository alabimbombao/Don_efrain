<template>
  <q-layout view="hHh lpR fFf" class="app-layout">
    <q-header elevated class="header-taller">
      <q-toolbar class="q-py-sm q-px-md">
        <div class="row items-center no-wrap">
          <q-avatar icon="build_circle" color="primary" text-color="white" size="38px" class="q-mr-sm shadow-1" />
          <div>
            <q-toolbar-title class="text-weight-bold text-h6 text-white leading-tight">
              Taller Don Efraín
            </q-toolbar-title>
            <div class="text-caption text-grey-4 text-weight-medium">
              Servicio técnico especializado en celulares y tablets
            </div>
          </div>
        </div>

        <q-space />

        <div class="search-header-box q-mx-md">
          <q-input
            v-model="textoBusqueda"
            dense
            outlined
            clearable
            placeholder="Buscar por cliente, marca, modelo, técnico o reparación..."
            class="header-search-input"
            bg-color="white"
          >
            <template v-slot:prepend>
              <q-icon name="search" color="grey-7" />
            </template>
          </q-input>
        </div>

        <q-space />

        <div class="row items-center q-gutter-sm">
          <q-btn
            unelevated
            color="white"
            text-color="primary"
            icon="add_circle"
            label="Nuevo servicio"
            class="text-weight-bold btn-nuevo shadow-2"
            @click="abrirModalNuevo()"
          />
        </div>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <q-page class="page-content q-pa-md">
        <div class="row q-col-gutter-md q-mb-md">
          <div class="col-6 col-sm-3">
            <q-card
              flat
              class="stat-card cursor-pointer"
              :class="{ 'stat-active': filtroEstado === 'en_taller' }"
              @click="toggleFiltroEspecial('en_taller')"
            >
              <q-card-section class="q-pa-sm text-center">
                <div class="row items-center justify-center q-gutter-xs text-caption text-grey-7">
                  <q-icon name="handyman" size="16px" color="primary" />
                  <span>En taller</span>
                </div>
                <div class="text-h5 text-weight-bolder text-primary q-mt-xs">
                  {{ stats.enTaller }}
                </div>
                <div class="text-caption text-grey-6" style="font-size: 12.5px">
                  Recibidos y en reparación
                </div>
              </q-card-section>
            </q-card>
          </div>

          <div class="col-6 col-sm-3">
            <q-card
              flat
              class="stat-card cursor-pointer"
              :class="{ 'stat-active': filtroEstado === 'listo' }"
              @click="toggleFiltroEspecial('listo')"
            >
              <q-card-section class="q-pa-sm text-center">
                <div class="row items-center justify-center q-gutter-xs text-caption text-grey-7">
                  <q-icon name="check_circle" size="16px" color="positive" />
                  <span>Listos para entregar</span>
                </div>
                <div class="text-h5 text-weight-bolder text-positive q-mt-xs">
                  {{ stats.listos }}
                </div>
                <div class="text-caption text-grey-6" style="font-size: 12.5px">
                  Esperando cliente
                </div>
              </q-card-section>
            </q-card>
          </div>

          <div class="col-6 col-sm-3">
            <q-card
              flat
              class="stat-card cursor-pointer"
              :class="{ 'stat-active': filtroPago === 'con_saldo' }"
              @click="toggleFiltroPago('con_saldo')"
            >
              <q-card-section class="q-pa-sm text-center">
                <div class="row items-center justify-center q-gutter-xs text-caption text-grey-7">
                  <q-icon name="warning" size="16px" color="negative" />
                  <span>Pagos pendientes / saldo</span>
                </div>
                <div class="text-h5 text-weight-bolder text-negative q-mt-xs">
                  {{ stats.conSaldoPendiente }}
                </div>
                <div class="text-caption text-grey-6" style="font-size: 12.5px">
                  Saldo fiado: ${{ formatearNumero(stats.totalSaldoPendiente) }}
                </div>
              </q-card-section>
            </q-card>
          </div>

          <div class="col-6 col-sm-3">
            <q-card flat class="stat-card">
              <q-card-section class="q-pa-sm text-center">
                <div class="row items-center justify-center q-gutter-xs text-caption text-grey-7">
                  <q-icon name="payments" size="16px" color="teal-8" />
                  <span>Total recaudado</span>
                </div>
                <div class="text-h5 text-weight-bolder text-teal-8 q-mt-xs">
                  ${{ formatearNumero(stats.totalRecaudado) }}
                </div>
                <div class="text-caption text-grey-6" style="font-size: 12.5px">
                  (Abonos: ${{ formatearNumero(stats.totalAbonado) }})
                </div>
              </q-card-section>
            </q-card>
          </div>
        </div>

        <!-- Barra de Control, Filtros y Herramientas -->
        <div class="control-bar q-pa-sm q-mb-md shadow-1">
          <div class="row items-center justify-between q-col-gutter-sm">
            <!-- Título y Conteo de Resultados -->
            <div class="col-12 col-md-auto row items-center q-gutter-sm">
              <div class="text-subtitle1 text-weight-bold text-grey-9">
                Servicios Registrados ({{ serviciosFiltrados.length }})
              </div>
              <q-badge color="indigo-1" text-color="indigo-9" class="q-px-md q-py-xs text-caption text-weight-bold shadow-1">
                Total en vista: ${{ formatearNumero(totalFiltrado) }}
              </q-badge>
              <q-badge v-if="filtroPago === 'con_saldo'" color="red-1" text-color="red-9" class="q-px-sm q-py-xs text-weight-bold">
                Filtro: Con saldo pendiente
                <q-icon name="close" class="cursor-pointer q-ml-xs" @click="filtroPago = 'todos'" />
              </q-badge>
            </div>

            <!-- Controles de Filtros, Orden y Vista -->
            <div class="col-12 col-md-auto row items-center q-gutter-sm justify-end">
              <!-- Selector de Etapa -->
              <div class="row items-center q-gutter-xs">
                <span class="text-caption text-grey-8 text-weight-medium">Etapa:</span>
                <q-select
                  v-model="filtroEstado"
                  :options="opcionesFiltroEtapas"
                  emit-value
                  map-options
                  dense
                  outlined
                  bg-color="white"
                  style="min-width: 175px"
                />
              </div>

              <!-- Selector de Orden -->
              <div class="row items-center q-gutter-xs">
                <span class="text-caption text-grey-8 text-weight-medium">Orden:</span>
                <q-select
                  v-model="criterioOrden"
                  :options="opcionesOrden"
                  emit-value
                  map-options
                  dense
                  outlined
                  bg-color="white"
                  style="min-width: 155px"
                />
              </div>


            </div>
          </div>

          <!-- Filtro rápido por chips de etapas -->
          <div class="row items-center q-gutter-xs q-mt-xs q-pt-xs border-top-subtle">
            <span class="text-caption text-grey-7 q-mr-xs">Filtro rápido:</span>
            <q-chip
              clickable
              dense
              :color="filtroEstado === 'todos' && filtroPago === 'todos' ? 'primary' : 'grey-2'"
              :text-color="filtroEstado === 'todos' && filtroPago === 'todos' ? 'white' : 'grey-8'"
              @click="limpiarFiltros()"
            >
              Todos ({{ servicios.length }})
            </q-chip>
            <q-chip
              clickable
              dense
              :color="filtroEstado === 'recibido' ? 'blue-grey' : 'grey-2'"
              :text-color="filtroEstado === 'recibido' ? 'white' : 'grey-8'"
              @click="filtroEstado = 'recibido'; filtroPago = 'todos'"
            >
              Recibidos ({{ contarPorEstado('recibido') }})
            </q-chip>
            <q-chip
              clickable
              dense
              :color="filtroEstado === 'en_reparacion' ? 'orange-9' : 'grey-2'"
              :text-color="filtroEstado === 'en_reparacion' ? 'white' : 'grey-8'"
              @click="filtroEstado = 'en_reparacion'; filtroPago = 'todos'"
            >
              En reparación ({{ contarPorEstado('en_reparacion') }})
            </q-chip>
            <q-chip
              clickable
              dense
              :color="filtroEstado === 'listo' ? 'positive' : 'grey-2'"
              :text-color="filtroEstado === 'listo' ? 'white' : 'grey-8'"
              @click="filtroEstado = 'listo'; filtroPago = 'todos'"
            >
              Listos ({{ contarPorEstado('listo') }})
            </q-chip>
            <q-chip
              clickable
              dense
              :color="filtroEstado === 'entregado' ? 'grey-8' : 'grey-2'"
              :text-color="filtroEstado === 'entregado' ? 'white' : 'grey-8'"
              @click="filtroEstado = 'entregado'; filtroPago = 'todos'"
            >
              Entregados ({{ contarPorEstado('entregado') }})
            </q-chip>
          </div>
        </div>

        <!-- ======================= VISTA: CUADRÍCULA DE TARJETAS ======================= -->
        <div class="row q-col-gutter-md">
          <div
            v-for="servicio in serviciosFiltrados"
            :key="servicio.id"
            class="col-12 col-sm-6 col-md-4"
          >
            <q-card flat class="device-card column justify-between full-height">
              <div>
                <!-- Cinta Superior de Estado de Pago -->
                <div
                  v-if="servicio.estadoPago === 'pendiente'"
                  class="ribbon-card bg-negative text-white text-caption text-center text-weight-bold q-py-xs"
                >
                  <q-icon name="error_outline" size="14px" class="q-mr-xs" />
                  PAGO PENDIENTE (SIN CANCELAR)
                </div>
                <div
                  v-else-if="servicio.estadoPago === 'abono'"
                  class="ribbon-card bg-orange-8 text-white text-caption text-center text-weight-bold q-py-xs"
                >
                  <q-icon name="hourglass_top" size="14px" class="q-mr-xs" />
                  ABONÓ: ${{ formatearNumero(servicio.montoAbonado || 0) }} — FALTA: ${{ formatearNumero(calcularSaldo(servicio)) }}
                </div>
                <div
                  v-else
                  class="ribbon-card bg-positive text-white text-caption text-center text-weight-bold q-py-xs"
                >
                  <q-icon name="verified" size="14px" class="q-mr-xs" />
                  PAGADO TOTALMENTE (CANCELADO)
                </div>

                <!-- Cabecera de la Tarjeta: Cliente y Dispositivo -->
                <q-card-section class="q-pb-xs">
                  <div class="row items-start justify-between no-wrap">
                    <div style="flex: 1; min-width: 0">
                      <div class="text-subtitle1 text-weight-bold text-grey-9 leading-tight ellipsis">
                        {{ servicio.cliente }}
                      </div>
                      <div class="text-body2 text-primary text-weight-medium row items-center q-gutter-xs q-mt-xs">
                        <q-icon name="smartphone" size="16px" />
                        <span class="ellipsis">{{ servicio.marca }} {{ servicio.modelo }}</span>
                      </div>
                      <div v-if="servicio.telefono" class="text-caption text-grey-6 row items-center q-gutter-xs q-mt-xs">
                        <q-icon name="phone" size="13px" />
                        <span>{{ servicio.telefono }}</span>
                      </div>
                    </div>

                    <!-- Badges de Reparaciones (Soporta múltiples reparaciones) -->
                    <div class="column items-end q-gutter-xs q-ml-sm">
                      <q-badge
                        v-for="(repTexto, idx) in obtenerListaReparaciones(servicio)"
                        :key="idx"
                        color="grey-2"
                        text-color="grey-9"
                        class="q-px-sm q-py-xs text-weight-medium shadow-1 text-right"
                      >
                        {{ repTexto }}
                      </q-badge>
                    </div>
                  </div>
                </q-card-section>

                <!-- Mini Stepper Visual de Etapas de Reparación -->
                <div class="q-px-md q-pt-xs q-pb-xs">
                  <div class="stepper-track row items-center justify-between">
                    <div
                      v-for="(etapa, index) in listaEtapasCiclo"
                      :key="etapa.valor"
                      class="step-item text-center"
                      :class="{
                        'cursor-pointer': servicio.estadoEquipo !== 'entregado',
                        'step-active': esEtapaActual(servicio.estadoEquipo, etapa.valor),
                        'step-completed': esEtapaCompletada(servicio.estadoEquipo, etapa.valor)
                      }"
                      @click="servicio.estadoEquipo !== 'entregado' && cambiarEtapaRapida(servicio, etapa.valor)"
                    >
                      <div class="step-dot">
                        <q-icon
                          v-if="esEtapaCompletada(servicio.estadoEquipo, etapa.valor) && !esEtapaActual(servicio.estadoEquipo, etapa.valor)"
                          name="check"
                          size="11px"
                        />
                        <span v-else>{{ index + 1 }}</span>
                      </div>
                      <div class="step-label">{{ etapa.etiqueta }}</div>
                    </div>
                  </div>
                </div>

                <q-separator class="q-my-xs" />

                <!-- Datos del Servicio -->
                <q-card-section class="q-py-xs q-gutter-xs text-caption">
                  <div class="row items-center justify-between q-my-xs">
                    <span class="text-grey-7">Estado actual:</span>
                    <q-badge :color="obtenerColorEstadoEquipo(servicio.estadoEquipo)" class="q-px-sm q-py-xs text-weight-bold">
                      <q-icon :name="obtenerIconoEstado(servicio.estadoEquipo)" size="13px" class="q-mr-xs" />
                      {{ obtenerEtiquetaEstadoEquipo(servicio.estadoEquipo) }}
                    </q-badge>
                  </div>

                  <div class="row items-center justify-between text-grey-8">
                    <span><strong>Técnico:</strong> {{ servicio.tecnico || 'No asignado' }}</span>
                    <span>{{ formatearFecha(servicio.fechaRecepcion) }}</span>
                  </div>

                  <div class="row items-center justify-between q-mt-xs text-subtitle2">
                    <span>Precio: <strong>${{ formatearNumero(servicio.precio) }}</strong></span>
                    <span class="text-caption text-grey-7">
                      <q-icon name="payment" size="13px" class="q-mr-xs" />
                      {{ obtenerEtiquetaPago(servicio.metodoPago) }}
                    </span>
                  </div>

                  <div v-if="servicio.observaciones" class="text-grey-7 q-mt-xs text-italic obs-box q-pa-xs">
                    <q-icon name="notes" size="13px" class="q-mr-xs text-grey-6" />
                    Obs: {{ servicio.observaciones }}
                  </div>

                  <!-- Advertencia de Bloqueo de Entrega si hay saldo -->
                  <div
                    v-if="servicio.estadoPago !== 'pagado' && servicio.estadoEquipo !== 'entregado'"
                    class="text-negative text-caption text-weight-bold q-mt-xs bg-red-1 q-pa-xs rounded-borders"
                    style="font-size: 12px"
                  >
                    <q-icon name="lock" size="13px" class="q-mr-xs" />
                    Entrega inhabilitada hasta cancelar saldo total
                  </div>

                  <!-- Calificación por Estrellas si ya fue entregado -->
                  <div v-if="servicio.estadoEquipo === 'entregado'" class="q-mt-xs row items-center justify-between bg-amber-1 q-pa-xs rounded-borders">
                    <span class="text-grey-8 text-weight-medium">Calificación del cliente:</span>
                    <q-rating
                      v-model="servicio.calificacion"
                      size="1.25em"
                      color="amber-8"
                      icon="star_border"
                      icon-selected="star"
                      @update:model-value="guardarCambiosDirectos()"
                    />
                  </div>
                </q-card-section>
              </div>

              <div>
                <q-separator />

                <!-- Acciones Rápidas y Menú de la Tarjeta -->
                <q-card-actions align="between" class="q-px-sm q-py-xs bg-grey-1">
                  <div class="row items-center q-gutter-xs">
                    <!-- Menú rápido para cambiar etapa (solo si no está entregado) -->
                    <q-btn-dropdown
                      v-if="servicio.estadoEquipo !== 'entregado'"
                      flat
                      dense
                      no-caps
                      size="sm"
                      color="primary"
                      label="Etapa"
                      icon="sync_alt"
                    >
                      <q-list dense style="min-width: 180px">
                        <q-item
                          clickable
                          v-close-popup
                          @click="cambiarEtapaRapida(servicio, 'recibido')"
                        >
                          <q-item-section>Recibido</q-item-section>
                        </q-item>
                        <q-item
                          clickable
                          v-close-popup
                          @click="cambiarEtapaRapida(servicio, 'en_reparacion')"
                        >
                          <q-item-section>En reparación</q-item-section>
                        </q-item>
                        <q-item
                          clickable
                          v-close-popup
                          @click="cambiarEtapaRapida(servicio, 'listo')"
                        >
                          <q-item-section>Listo para entregar</q-item-section>
                        </q-item>
                        <q-item
                          clickable
                          v-close-popup
                          :disable="servicio.estadoPago !== 'pagado'"
                          @click="cambiarEtapaRapida(servicio, 'entregado')"
                        >
                          <q-item-section>
                            <q-item-label>Entregado</q-item-label>
                            <q-item-label v-if="servicio.estadoPago !== 'pagado'" caption class="text-negative">
                              Requiere pago completo
                            </q-item-label>
                          </q-item-section>
                        </q-item>
                      </q-list>
                    </q-btn-dropdown>
                  </div>

                  <div class="row items-center q-gutter-xs">
                    <q-btn
                      v-if="servicio.estadoEquipo !== 'entregado'"
                      flat
                      dense
                      icon="edit"
                      color="primary"
                      label="Editar"
                      no-caps
                      size="sm"
                      @click="abrirModalEditar(servicio)"
                    />
                    <q-btn
                      v-if="servicio.estadoEquipo !== 'entregado'"
                      flat
                      dense
                      icon="delete"
                      color="negative"
                      label="Eliminar"
                      no-caps
                      size="sm"
                      @click="abrirConfirmarEliminar(servicio)"
                    />
                  </div>
                </q-card-actions>
              </div>
            </q-card>
          </div>

          <!-- Estado Vacío -->
          <div v-if="serviciosFiltrados.length === 0" class="col-12 text-center q-pa-xl empty-box">
            <q-icon name="search_off" size="48px" color="grey-5" class="q-mb-sm" />
            <div class="text-subtitle1 text-grey-8 text-weight-medium">
              No hay servicios registrados con los filtros aplicados
            </div>
            <div class="text-caption text-grey-6 q-mb-md">
              Intenta cambiar los términos de búsqueda o restablecer los filtros.
            </div>
            <q-btn
              unelevated
              color="primary"
              label="Limpiar búsqueda y filtros"
              no-caps
              icon="refresh"
              @click="limpiarFiltros()"
            />
          </div>
        </div>



        <!-- Botón Flotante Agregar -->
        <q-page-sticky position="bottom-right" :offset="[20, 20]">
          <q-btn fab icon="add" color="primary" class="shadow-4" @click="abrirModalNuevo()">
            <q-tooltip anchor="center left" self="center right">Registrar nuevo servicio</q-tooltip>
          </q-btn>
        </q-page-sticky>
      </q-page>
    </q-page-container>

    <!-- Modal Formulario (Crear / Editar) -->
    <q-dialog v-model="mostrarModalFormulario" persistent>
      <q-card style="width: 700px; max-width: 95vw" class="modal-form-card shadow-5">
        <q-card-section class="row items-center bg-primary text-white q-py-sm">
          <q-icon :name="modoEdicion ? 'edit_note' : 'post_add'" size="24px" class="q-mr-sm" />
          <div class="text-h6 text-weight-bold">
            {{ modoEdicion ? 'Editar Servicio Técnico' : 'Registrar Nuevo Servicio' }}
          </div>
          <q-space />
          <q-btn flat round dense icon="close" text-color="white" @click="cerrarModalFormulario()" />
        </q-card-section>

        <q-card-section style="max-height: 75vh" class="scroll bg-white text-grey-9 q-pa-md">
          <q-form ref="formularioRef" class="q-gutter-y-sm">
            <div class="text-caption text-grey-7 text-weight-bold q-mb-xs">
              DATOS DEL CLIENTE Y DISPOSITIVO
            </div>
            <div class="row q-col-gutter-sm">
              <div class="col-12 col-sm-6">
                <q-input
                  v-model="formulario.cliente"
                  label="Nombre del cliente *"
                  outlined
                  dense
                  bg-color="white"
                  :rules="[val => (val && val.trim().length > 0) || 'El nombre del cliente es obligatorio']"
                />
              </div>
              <div class="col-12 col-sm-6">
                <q-input
                  v-model="formulario.telefono"
                  label="Teléfono / Celular (WhatsApp)"
                  outlined
                  dense
                  bg-color="white"
                  placeholder="Ej: 3001234567"
                  hint="Opcional para enviar notificaciones"
                />
              </div>

              <!-- Selector de Marcas (20 marcas solicitadas + Otra) -->
              <div :class="formulario.marca === 'Otra' ? 'col-6 col-sm-3' : 'col-6 col-sm-6'">
                <q-select
                  v-model="formulario.marca"
                  :options="opcionesMarcas"
                  label="Marca del equipo *"
                  outlined
                  dense
                  bg-color="white"
                  :rules="[val => !!val || 'Selecciona una marca']"
                />
              </div>
              <div class="col-6 col-sm-3" v-if="formulario.marca === 'Otra'">
                <q-input
                  v-model="formulario.otraMarca"
                  label="Especifique marca *"
                  outlined
                  dense
                  bg-color="white"
                  :rules="[val => (val && val.trim().length > 0) || 'Especifique la marca']"
                />
              </div>

              <div :class="formulario.marca === 'Otra' ? 'col-12 col-sm-6' : 'col-6 col-sm-6'">
                <q-input
                  v-model="formulario.modelo"
                  label="Modelo del equipo *"
                  outlined
                  dense
                  bg-color="white"
                  placeholder="Ej: Galaxy A54, iPhone 12, etc."
                  :rules="[val => (val && val.trim().length > 0) || 'Obligatorio']"
                />
              </div>

              <!-- Pregunta obligatoria: ¿Cuántas reparaciones se van a hacer? -->
              <div class="col-12 col-sm-6">
                <q-select
                  v-model="formulario.cantidadReparaciones"
                  :options="opcionesCantidadReparaciones"
                  label="¿Cuántas reparaciones se van a hacer? *"
                  outlined
                  dense
                  bg-color="white"
                  emit-value
                  map-options
                  :rules="[val => !!val && val >= 1 || 'Indica la cantidad']"
                  @update:model-value="ajustarCantidadReparaciones"
                >
                  <template v-slot:prepend>
                    <q-icon name="format_list_numbered" size="18px" color="primary" />
                  </template>
                </q-select>
              </div>

              <div class="col-12 col-sm-6">
                <q-select
                  v-model="formulario.tecnico"
                  :options="opcionesTecnicos"
                  label="Técnico asignado *"
                  outlined
                  dense
                  bg-color="white"
                  :rules="[val => !!val || 'Selecciona un técnico']"
                />
              </div>

              <div class="col-12">
                <q-input
                  v-model="formulario.fechaRecepcion"
                  type="datetime-local"
                  label="Fecha y hora de recepción (fija) *"
                  outlined
                  dense
                  bg-color="grey-1"
                  readonly
                  stack-label
                  hint="Fijada automáticamente al iniciar el registro (no modificable)"
                  :rules="[val => !!val || 'La fecha es obligatoria']"
                >
                  <template v-slot:prepend>
                    <q-icon name="lock" color="grey-6" size="18px" />
                  </template>
                </q-input>
              </div>
            </div>

            <!-- Sección Dinámica: Selección de Tipos de Reparaciones según la cantidad elegida -->
            <div class="q-mt-sm bg-blue-50 q-pa-sm rounded-borders border-subtle">
              <div class="text-caption text-primary text-weight-bold q-mb-xs row items-center">
                <q-icon name="build" size="15px" class="q-mr-xs" />
                TIPOS DE REPARACIÓN A REALIZAR ({{ (formulario.reparaciones || []).length }})
              </div>
              <div class="row q-col-gutter-sm">
                <div
                  v-for="(rep, index) in formulario.reparaciones"
                  :key="index"
                  class="col-12"
                >
                  <div class="row q-col-gutter-sm items-center">
                    <div :class="rep.tipo === 'otros' ? 'col-12 col-sm-6' : 'col-12'">
                      <q-select
                        v-model="rep.tipo"
                        :options="opcionesTipoReparacion"
                        emit-value
                        map-options
                        :label="`Reparación #${index + 1} *`"
                        outlined
                        dense
                        bg-color="white"
                        :rules="[val => !!val || `Selecciona la reparación #${index + 1}`]"
                      >
                        <template v-slot:prepend>
                          <q-icon :name="obtenerIconoReparacion(rep.tipo)" size="18px" color="primary" />
                        </template>
                      </q-select>
                    </div>
                    <div class="col-12 col-sm-6" v-if="rep.tipo === 'otros'">
                      <q-input
                        v-model="rep.especificacionOtros"
                        :label="`Especificar reparación #${index + 1} (Otros) *`"
                        outlined
                        dense
                        bg-color="white"
                        placeholder="Ej: Cambio de cámara, botón..."
                        :rules="[val => (val && val.trim().length > 0) || 'Especifica la reparación']"
                      />
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <q-separator class="q-my-sm" />

            <div class="text-caption text-grey-7 text-weight-bold q-mb-xs">
              CONDICIONES ECONÓMICAS Y PAGO
            </div>
            <div class="row q-col-gutter-sm">
              <div class="col-6 col-sm-4">
                <q-input
                  v-model.number="formulario.precio"
                  type="number"
                  label="Precio total cobrado *"
                  outlined
                  dense
                  bg-color="white"
                  prefix="$"
                  :rules="[val => val !== null && val >= 0 || 'Ingresa un precio válido']"
                  @update:model-value="ajustarAbonoSegunPrecio()"
                />
              </div>
              <div class="col-6 col-sm-4">
                <q-select
                  v-model="formulario.metodoPago"
                  :options="opcionesMetodoPago"
                  label="Método de pago *"
                  outlined
                  dense
                  bg-color="white"
                  :rules="[val => !!val || 'Selecciona un método']"
                />
              </div>
              <div class="col-12 col-sm-4">
                <q-select
                  v-model="formulario.estadoPago"
                  :options="opcionesEstadoPago"
                  emit-value
                  map-options
                  label="Estado del pago *"
                  outlined
                  dense
                  bg-color="white"
                  :rules="[val => !!val || 'Selecciona el estado del pago']"
                  @update:model-value="alCambiarEstadoPago()"
                />
              </div>

              <div class="col-12" v-if="formulario.estadoPago === 'abono'">
                <q-input
                  v-model.number="formulario.montoAbonado"
                  type="number"
                  label="Monto abonado por el cliente *"
                  outlined
                  dense
                  bg-color="white"
                  prefix="$"
                  :hint="`Saldo restante por pagar: $${formatearNumero(Math.max((formulario.precio || 0) - (formulario.montoAbonado || 0), 0))}`"
                  :rules="[
                    val => val !== null && val >= 0 || 'Debe ser un valor mayor o igual a 0',
                    val => val <= formulario.precio || 'El abono no puede superar el precio cobrado'
                  ]"
                />
              </div>
            </div>

            <q-separator class="q-my-sm" />

            <div class="text-caption text-grey-7 text-weight-bold q-mb-xs">
              ETAPA DE TRABAJO Y DETALLES TÉCNICOS
            </div>
            <div class="row q-col-gutter-sm">
              <div class="col-12">
                <q-select
                  v-model="formulario.estadoEquipo"
                  :options="obtenerOpcionesEstadoEquipoFormulario()"
                  emit-value
                  map-options
                  label="Estado del equipo (Etapa) *"
                  outlined
                  dense
                  :bg-color="!modoEdicion ? 'grey-1' : 'white'"
                  :readonly="!modoEdicion"
                  :hint="!modoEdicion ? 'El registro nuevo inicia obligatoriamente en etapa Recibido' : undefined"
                  :rules="[val => !!val || 'Selecciona la etapa']"
                >
                  <template v-if="!modoEdicion" v-slot:prepend>
                    <q-icon name="lock" color="grey-6" size="18px" />
                  </template>
                </q-select>
              </div>

              <div class="col-12">
                <q-input
                  v-model="formulario.observaciones"
                  type="textarea"
                  label="Observaciones y diagnóstico inicial (opcional)"
                  outlined
                  dense
                  bg-color="white"
                  autogrow
                  hint="Ej: pantalla partida en la esquina superior, equipo apagado, incluye tarjeta SIM"
                />
              </div>
            </div>
          </q-form>
        </q-card-section>

        <q-card-actions align="right" class="q-pa-md bg-grey-1">
          <q-btn flat label="Cancelar" color="grey-8" @click="cerrarModalFormulario()" />
          <q-btn unelevated label="Guardar servicio" color="primary" icon="save" @click="guardarServicio()" />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <!-- Modal Confirmación de Eliminación -->
    <q-dialog v-model="mostrarModalEliminar">
      <q-card style="width: 420px; max-width: 90vw" class="rounded-borders">
        <q-card-section class="row items-center bg-negative text-white q-py-sm">
          <q-icon name="warning" size="24px" class="q-mr-sm" />
          <div class="text-h6 text-weight-bold">¿Eliminar servicio?</div>
        </q-card-section>
        <q-card-section class="text-body2 text-grey-8 q-pa-md bg-white">
          Vas a eliminar permanentemente el registro de
          <strong>{{ servicioAEliminar ? servicioAEliminar.cliente : '' }}</strong>
          ({{ servicioAEliminar ? servicioAEliminar.marca + ' ' + servicioAEliminar.modelo : '' }}).
          <div class="q-mt-sm text-caption text-negative text-weight-medium">
            Esta acción no se puede deshacer.
          </div>
        </q-card-section>
        <q-card-actions align="right" class="bg-grey-1 q-pa-md">
          <q-btn flat label="Cancelar" color="grey-8" @click="mostrarModalEliminar = false" />
          <q-btn unelevated label="Sí, eliminar" color="negative" icon="delete" @click="eliminarServicio()" />
        </q-card-actions>
      </q-card>
    </q-dialog>



    <!-- Notificación Flotante Estilo Toast -->
    <transition name="fade">
      <div
        v-if="mensajeAviso"
        class="toast-notificacion shadow-4"
        :class="mensajeAvisoTipo === 'error' ? 'toast-error' : 'toast-exito'"
      >
        <q-icon :name="mensajeAvisoTipo === 'error' ? 'cancel' : 'check_circle'" size="20px" class="q-mr-sm" />
        <span>{{ mensajeAviso }}</span>
      </div>
    </transition>
  </q-layout>
</template>

<script setup>
import { ref, computed } from 'vue'
import { useLocalStorage } from '@vueuse/core'

// Lista de marcas solicitada por el usuario + opción Otra
const opcionesMarcas = [
  'Samsung',
  'Apple',
  'Xiaomi',
  'Motorola',
  'Huawei',
  'Google',
  'OnePlus',
  'OPPO',
  'Vivo',
  'Realme',
  'Nokia',
  'Sony',
  'ASUS',
  'Honor',
  'ZTE',
  'Lenovo',
  'TCL',
  'Nothing',
  'Tecno',
  'Infinix',
  'Otra'
]

// Opciones de cantidad de reparaciones
const opcionesCantidadReparaciones = [1, 2, 3, 4, 5, 6]

// Tipos de reparación disponibles
const opcionesTipoReparacion = [
  { label: 'Cambio de pantalla', value: 'pantalla' },
  { label: 'Cambio de batería', value: 'bateria' },
  { label: 'Cambio de pin de carga', value: 'pin_carga' },
  { label: 'Liberación', value: 'liberacion' },
  { label: 'Mantenimiento de software', value: 'software' },
  { label: 'Cambio de flex', value: 'flex' },
  { label: 'Otros', value: 'otros' },
]

// Datos iniciales de demostración en caso de que localStorage esté vacío
const datosIniciales = [
  {
    id: '1709901000001',
    cliente: 'Carlos Mendoza',
    telefono: '3104567890',
    marca: 'Samsung',
    modelo: 'Galaxy A54',
    cantidadReparaciones: 2,
    reparaciones: [
      { tipo: 'pantalla', especificacionOtros: '' },
      { tipo: 'bateria', especificacionOtros: '' }
    ],
    tipoReparacion: 'pantalla',
    especificacionOtros: '',
    tecnico: 'Don Efraín',
    fechaRecepcion: '2026-03-08T09:30',
    precio: 220000,
    metodoPago: 'Transferencia',
    estadoPago: 'abono',
    montoAbonado: 100000,
    estadoEquipo: 'en_reparacion',
    calificacion: 0,
    observaciones: 'Vidrio roto en esquina derecha y batería inflada.',
  },
  {
    id: '1709901000002',
    cliente: 'Mariana Gómez',
    telefono: '3157891234',
    marca: 'Apple',
    modelo: 'iPhone 11',
    cantidadReparaciones: 1,
    reparaciones: [
      { tipo: 'bateria', especificacionOtros: '' }
    ],
    tipoReparacion: 'bateria',
    especificacionOtros: '',
    tecnico: 'Técnico 1',
    fechaRecepcion: '2026-03-07T14:15',
    precio: 160000,
    metodoPago: 'Efectivo',
    estadoPago: 'pagado',
    montoAbonado: 160000,
    estadoEquipo: 'listo',
    calificacion: 0,
    observaciones: 'Condición de batería en 74%. Desea repuesto original.',
  },
  {
    id: '1709901000003',
    cliente: 'Andrés Felipe Restrepo',
    telefono: '3209876543',
    marca: 'Xiaomi',
    modelo: 'Redmi Note 12',
    cantidadReparaciones: 1,
    reparaciones: [
      { tipo: 'pin_carga', especificacionOtros: '' }
    ],
    tipoReparacion: 'pin_carga',
    especificacionOtros: '',
    tecnico: 'Técnico 2',
    fechaRecepcion: '2026-03-08T11:00',
    precio: 75000,
    metodoPago: 'Efectivo',
    estadoPago: 'pendiente',
    montoAbonado: 0,
    estadoEquipo: 'recibido',
    calificacion: 0,
    observaciones: 'No agarra carga rápida, puerto flojo.',
  },
  {
    id: '1709901000004',
    cliente: 'Patricia Quintero',
    telefono: '3123456789',
    marca: 'Motorola',
    modelo: 'Moto G52',
    cantidadReparaciones: 1,
    reparaciones: [
      { tipo: 'software', especificacionOtros: '' }
    ],
    tipoReparacion: 'software',
    especificacionOtros: '',
    tecnico: 'Don Efraín',
    fechaRecepcion: '2026-03-06T16:45',
    precio: 50000,
    metodoPago: 'Tarjeta',
    estadoPago: 'pagado',
    montoAbonado: 50000,
    estadoEquipo: 'entregado',
    calificacion: 5,
    observaciones: 'Reinicio constante en el logo. Se realizó flasheo limpio.',
  },
]

const opcionesTecnicos = ['Don Efraín', 'Técnico 1', 'Técnico 2']
const opcionesMetodoPago = ['Efectivo', 'Transferencia', 'Tarjeta']

const opcionesEstadoPago = [
  { label: 'Pagado totalmente', value: 'pagado' },
  { label: 'Abono parcial', value: 'abono' },
  { label: 'Pendiente de pago', value: 'pendiente' },
]

const opcionesFiltroEtapas = [
  { label: 'Todas las etapas', value: 'todos' },
  { label: 'Recibidos', value: 'recibido' },
  { label: 'En reparación', value: 'en_reparacion' },
  { label: 'Listos para entregar', value: 'listo' },
  { label: 'Entregados', value: 'entregado' },
]

const opcionesOrden = [
  { label: 'Más recientes', value: 'recientes' },
  { label: 'Más antiguos', value: 'antiguos' },
  { label: 'Mayor precio', value: 'precio_desc' },
  { label: 'Menor precio', value: 'precio_asc' },
  { label: 'Cliente (A-Z)', value: 'cliente_asc' },
]

const listaEtapasCiclo = [
  { valor: 'recibido', etiqueta: 'Recibido' },
  { valor: 'en_reparacion', etiqueta: 'Reparación' },
  { valor: 'listo', etiqueta: 'Listo' },
  { valor: 'entregado', etiqueta: 'Entregado' },
]

// Estado reactivo persistente
const servicios = useLocalStorage('taller-don-efrain-servicios', datosIniciales)

// Estado local de la interfaz
const mostrarModalFormulario = ref(false)
const mostrarModalEliminar = ref(false)
const modoEdicion = ref(false)
const idEnEdicion = ref(null)
const servicioAEliminar = ref(null)
const formularioRef = ref(null)

const textoBusqueda = ref('')
const filtroEstado = ref('todos')
const filtroPago = ref('todos')
const criterioOrden = ref('recientes')

const mensajeAviso = ref('')
const mensajeAvisoTipo = ref('exito')

const formularioVacio = () => ({
  cliente: '',
  telefono: '',
  marca: null,
  otraMarca: '',
  modelo: '',
  cantidadReparaciones: 1,
  reparaciones: [
    { tipo: null, especificacionOtros: '' }
  ],
  tipoReparacion: null,
  especificacionOtros: '',
  tecnico: 'Don Efraín',
  fechaRecepcion: obtenerFechaHoraActual(),
  precio: 0,
  metodoPago: 'Efectivo',
  estadoPago: 'pendiente',
  montoAbonado: 0,
  estadoEquipo: 'recibido',
  calificacion: 0,
  observaciones: '',
})

const formulario = ref(formularioVacio())

// ================= COMPUTED PROPERTIES (OPTIMIZACIÓN VUE 3) =================

const stats = computed(() => {
  const lista = servicios.value || []
  const enTaller = lista.filter(s => s.estadoEquipo === 'recibido' || s.estadoEquipo === 'en_reparacion').length
  const listos = lista.filter(s => s.estadoEquipo === 'listo').length
  const entregados = lista.filter(s => s.estadoEquipo === 'entregado').length

  const pendientes = lista.filter(s => s.estadoPago === 'pendiente').length
  const conSaldoPendiente = lista.filter(s => s.estadoPago === 'pendiente' || (s.estadoPago === 'abono' && calcularSaldo(s) > 0)).length

  const totalAbonado = lista
    .filter(s => s.estadoPago === 'abono')
    .reduce((total, s) => total + (Number(s.montoAbonado) || 0), 0)

  const totalSaldoPendiente = lista.reduce((total, s) => {
    if (s.estadoPago === 'pendiente') return total + (Number(s.precio) || 0)
    if (s.estadoPago === 'abono') return total + calcularSaldo(s)
    return total
  }, 0)

  const totalRecaudado = lista.reduce((total, s) => {
    if (s.estadoPago === 'pagado') return total + (Number(s.precio) || 0)
    if (s.estadoPago === 'abono') return total + (Number(s.montoAbonado) || 0)
    return total
  }, 0)

  return {
    enTaller,
    listos,
    entregados,
    pendientes,
    conSaldoPendiente,
    totalAbonado,
    totalSaldoPendiente,
    totalRecaudado,
  }
})

const serviciosFiltrados = computed(() => {
  let resultado = [...(servicios.value || [])]

  // Filtro por etapa
  if (filtroEstado.value === 'en_taller') {
    resultado = resultado.filter(s => s.estadoEquipo === 'recibido' || s.estadoEquipo === 'en_reparacion')
  } else if (filtroEstado.value !== 'todos') {
    resultado = resultado.filter(s => s.estadoEquipo === filtroEstado.value)
  }

  // Filtro especial de pago
  if (filtroPago.value === 'con_saldo') {
    resultado = resultado.filter(s => s.estadoPago === 'pendiente' || (s.estadoPago === 'abono' && calcularSaldo(s) > 0))
  }

  // Filtro por texto de búsqueda
  if (textoBusqueda.value && textoBusqueda.value.trim().length > 0) {
    const texto = textoBusqueda.value.trim().toLowerCase()
    resultado = resultado.filter(s => {
      const cliente = (s.cliente || '').toLowerCase()
      const marca = (s.marca || '').toLowerCase()
      const modelo = (s.modelo || '').toLowerCase()
      const tecnico = (s.tecnico || '').toLowerCase()
      const telefono = (s.telefono || '').toLowerCase()
      const reparacionesTexto = obtenerListaReparaciones(s).join(' ').toLowerCase()

      return (
        cliente.includes(texto) ||
        marca.includes(texto) ||
        modelo.includes(texto) ||
        tecnico.includes(texto) ||
        telefono.includes(texto) ||
        reparacionesTexto.includes(texto)
      )
    })
  }

  // Ordenamiento
  if (criterioOrden.value === 'recientes') {
    resultado.sort((a, b) => new Date(b.fechaRecepcion || 0) - new Date(a.fechaRecepcion || 0))
  } else if (criterioOrden.value === 'antiguos') {
    resultado.sort((a, b) => new Date(a.fechaRecepcion || 0) - new Date(b.fechaRecepcion || 0))
  } else if (criterioOrden.value === 'precio_desc') {
    resultado.sort((a, b) => (Number(b.precio) || 0) - (Number(a.precio) || 0))
  } else if (criterioOrden.value === 'precio_asc') {
    resultado.sort((a, b) => (Number(a.precio) || 0) - (Number(b.precio) || 0))
  } else if (criterioOrden.value === 'cliente_asc') {
    resultado.sort((a, b) => (a.cliente || '').localeCompare(b.cliente || ''))
  }

  return resultado
})

const totalFiltrado = computed(() => {
  return serviciosFiltrados.value.reduce((total, s) => total + (Number(s.precio) || 0), 0)
})

// ================= GESTIÓN DINÁMICA DE REPARACIONES Y MARCAS =================

function ajustarCantidadReparaciones(nuevaCantidad) {
  const cant = Math.max(Number(nuevaCantidad) || 1, 1)
  if (!formulario.value.reparaciones) {
    formulario.value.reparaciones = []
  }
  while (formulario.value.reparaciones.length < cant) {
    formulario.value.reparaciones.push({ tipo: null, especificacionOtros: '' })
  }
  if (formulario.value.reparaciones.length > cant) {
    formulario.value.reparaciones = formulario.value.reparaciones.slice(0, cant)
  }
}

function obtenerOpcionesEstadoEquipoFormulario() {
  if (!modoEdicion.value) {
    return [
      { label: 'Recibido (Etapa inicial de registro)', value: 'recibido' }
    ]
  }
  const esPagoIncompleto = formulario.value.estadoPago === 'pendiente' || formulario.value.estadoPago === 'abono'
  return [
    { label: 'Recibido', value: 'recibido' },
    { label: 'En reparación', value: 'en_reparacion' },
    { label: 'Listo para entregar', value: 'listo' },
    {
      label: esPagoIncompleto ? 'Entregado (Inhabilitado: Requiere pago completo)' : 'Entregado',
      value: 'entregado',
      disable: esPagoIncompleto,
    },
  ]
}

function alCambiarEstadoPago() {
  if (formulario.value.estadoPago === 'pagado') {
    formulario.value.montoAbonado = formulario.value.precio
  } else if (formulario.value.estadoPago === 'pendiente') {
    formulario.value.montoAbonado = 0
    if (formulario.value.estadoEquipo === 'entregado') {
      formulario.value.estadoEquipo = 'listo'
      mostrarAviso('Se cambió el estado a "Listo para entregar" porque tiene saldo pendiente.', 'error')
    }
  } else if (formulario.value.estadoPago === 'abono') {
    if (formulario.value.estadoEquipo === 'entregado') {
      formulario.value.estadoEquipo = 'listo'
      mostrarAviso('Se cambió el estado a "Listo para entregar" porque tiene saldo pendiente.', 'error')
    }
  }
}

function ajustarAbonoSegunPrecio() {
  if (formulario.value.estadoPago === 'pagado') {
    formulario.value.montoAbonado = formulario.value.precio
  }
}

function abrirModalNuevo() {
  modoEdicion.value = false
  idEnEdicion.value = null
  formulario.value = formularioVacio()
  formulario.value.fechaRecepcion = obtenerFechaHoraActual()
  mostrarModalFormulario.value = true
}

function abrirModalEditar(servicio) {
  modoEdicion.value = true
  idEnEdicion.value = servicio.id
  const copia = JSON.parse(JSON.stringify(servicio))

  // Normalizar marca
  if (copia.marca && !opcionesMarcas.includes(copia.marca)) {
    copia.otraMarca = copia.marca
    copia.marca = 'Otra'
  } else {
    copia.otraMarca = ''
  }

  // Normalizar reparaciones múltiples (retrocompatibilidad)
  if (!copia.reparaciones || !Array.isArray(copia.reparaciones) || copia.reparaciones.length === 0) {
    copia.reparaciones = [
      { tipo: copia.tipoReparacion || null, especificacionOtros: copia.especificacionOtros || '' }
    ]
    copia.cantidadReparaciones = 1
  } else {
    copia.cantidadReparaciones = copia.reparaciones.length
  }

  formulario.value = copia
  mostrarModalFormulario.value = true
}

function cerrarModalFormulario() {
  mostrarModalFormulario.value = false
  formulario.value = formularioVacio()
}

function guardarServicio() {
  formularioRef.value.validate().then((esValido) => {
    if (!esValido) {
      mostrarAviso('Revisa los campos obligatorios marcados en rojo', 'error')
      return
    }

    // Regla de negocio inquebrantable
    if (formulario.value.estadoEquipo === 'entregado' && formulario.value.estadoPago !== 'pagado') {
      mostrarAviso('¡Imposible entregar! Un equipo no se puede marcar como Entregado si no está totalmente Pagado.', 'error')
      return
    }

    // Normalizar marca
    let marcaFinal = formulario.value.marca
    if (marcaFinal === 'Otra') {
      marcaFinal = (formulario.value.otraMarca || '').trim() || 'Otra'
    }

    // Normalizar y limpiar reparaciones
    const reparacionesFinales = (formulario.value.reparaciones || []).map(r => ({
      tipo: r.tipo,
      especificacionOtros: r.tipo === 'otros' ? (r.especificacionOtros || '').trim() : ''
    }))

    const primerTipo = reparacionesFinales[0] ? reparacionesFinales[0].tipo : null
    const primerOtros = reparacionesFinales[0] ? reparacionesFinales[0].especificacionOtros : ''

    if (formulario.value.estadoPago === 'pagado') {
      formulario.value.montoAbonado = Number(formulario.value.precio) || 0
    } else if (formulario.value.estadoPago === 'pendiente') {
      formulario.value.montoAbonado = 0
    }

    if (!modoEdicion.value) {
      formulario.value.estadoEquipo = 'recibido'
    }

    const payload = {
      ...formulario.value,
      estadoEquipo: !modoEdicion.value ? 'recibido' : formulario.value.estadoEquipo,
      marca: marcaFinal,
      reparaciones: reparacionesFinales,
      cantidadReparaciones: reparacionesFinales.length,
      tipoReparacion: primerTipo,
      especificacionOtros: primerOtros
    }

    if (modoEdicion.value) {
      const indice = servicios.value.findIndex(s => s.id === idEnEdicion.value)
      if (indice !== -1) {
        servicios.value[indice] = { ...payload, id: idEnEdicion.value }
      }
      mostrarAviso('Servicio actualizado correctamente')
    } else {
      const nuevoServicio = {
        ...payload,
        id: Date.now().toString(),
      }
      servicios.value.unshift(nuevoServicio)
      mostrarAviso('Servicio registrado exitosamente')
    }

    cerrarModalFormulario()
  })
}

function cambiarEtapaRapida(servicio, nuevaEtapa) {
  if (nuevaEtapa === 'entregado' && servicio.estadoPago !== 'pagado') {
    mostrarAviso('No se puede marcar como Entregado hasta que el cliente cancele el saldo total.', 'error')
    return
  }

  const indice = servicios.value.findIndex(s => s.id === servicio.id)
  if (indice !== -1) {
    servicios.value[indice].estadoEquipo = nuevaEtapa
    mostrarAviso(`Etapa actualizada a "${obtenerEtiquetaEstadoEquipo(nuevaEtapa)}"`)
  }
}

function guardarCambiosDirectos() {
  mostrarAviso('Calificación guardada correctamente')
}

function abrirConfirmarEliminar(servicio) {
  servicioAEliminar.value = servicio
  mostrarModalEliminar.value = true
}

function eliminarServicio() {
  if (!servicioAEliminar.value) return
  servicios.value = servicios.value.filter(s => s.id !== servicioAEliminar.value.id)
  mostrarModalEliminar.value = false
  mostrarAviso('Servicio eliminado')
  servicioAEliminar.value = null
}

function toggleFiltroEspecial(etapa) {
  if (filtroEstado.value === etapa) {
    filtroEstado.value = 'todos'
  } else {
    filtroEstado.value = etapa
  }
  filtroPago.value = 'todos'
}

function toggleFiltroPago(tipo) {
  if (filtroPago.value === tipo) {
    filtroPago.value = 'todos'
  } else {
    filtroPago.value = tipo
    filtroEstado.value = 'todos'
  }
}

function limpiarFiltros() {
  textoBusqueda.value = ''
  filtroEstado.value = 'todos'
  filtroPago.value = 'todos'
  criterioOrden.value = 'recientes'
}



// Helpers de conteo y cálculo
function contarPorEstado(estado) {
  return (servicios.value || []).filter(s => s.estadoEquipo === estado).length
}

function calcularSaldo(servicio) {
  if (!servicio) return 0
  if (servicio.estadoPago === 'pagado') return 0
  const precio = Number(servicio.precio) || 0
  if (servicio.estadoPago === 'pendiente') return precio
  const abonado = Number(servicio.montoAbonado) || 0
  return Math.max(precio - abonado, 0)
}

function esEtapaActual(etapaActual, etapaEvaluar) {
  return etapaActual === etapaEvaluar
}

function esEtapaCompletada(etapaActual, etapaEvaluar) {
  const orden = ['recibido', 'en_reparacion', 'listo', 'entregado']
  const indiceActual = orden.indexOf(etapaActual)
  const indiceEvaluar = orden.indexOf(etapaEvaluar)
  return indiceActual >= indiceEvaluar
}

function obtenerColorEstadoEquipo(estado) {
  if (estado === 'recibido') return 'blue-grey'
  if (estado === 'en_reparacion') return 'orange-9'
  if (estado === 'listo') return 'positive'
  if (estado === 'entregado') return 'grey-7'
  return 'grey'
}

function obtenerIconoEstado(estado) {
  if (estado === 'recibido') return 'inventory_2'
  if (estado === 'en_reparacion') return 'build'
  if (estado === 'listo') return 'done_all'
  if (estado === 'entregado') return 'task_alt'
  return 'info'
}

function obtenerIconoReparacion(tipo) {
  if (tipo === 'pantalla') return 'smartphone'
  if (tipo === 'bateria') return 'battery_charging_full'
  if (tipo === 'pin_carga') return 'power'
  if (tipo === 'liberacion') return 'lock_open'
  if (tipo === 'software') return 'terminal'
  if (tipo === 'flex') return 'cable'
  return 'devices'
}

function obtenerEtiquetaEstadoEquipo(estado) {
  if (estado === 'recibido') return 'Recibido'
  if (estado === 'en_reparacion') return 'En reparación'
  if (estado === 'listo') return 'Listo para entregar'
  if (estado === 'entregado') return 'Entregado'
  return estado
}

function obtenerListaReparaciones(servicio) {
  if (!servicio) return ['—']
  if (servicio.reparaciones && Array.isArray(servicio.reparaciones) && servicio.reparaciones.length > 0) {
    return servicio.reparaciones.map(r => {
      if (r.tipo === 'otros') {
        return r.especificacionOtros ? `Otros: ${r.especificacionOtros}` : 'Otros'
      }
      const encontrado = opcionesTipoReparacion.find(o => o.value === r.tipo)
      return encontrado ? encontrado.label : (r.tipo || '—')
    })
  }

  // Fallback para servicios guardados en versiones anteriores
  const valor = typeof servicio === 'object' ? servicio.tipoReparacion : servicio
  const especificacion = typeof servicio === 'object' ? servicio.especificacionOtros : null

  if (valor === 'otros') {
    return [especificacion ? `Otros: ${especificacion}` : 'Otros']
  }
  const encontrado = opcionesTipoReparacion.find(o => o.value === valor)
  return [encontrado ? encontrado.label : (valor || '—')]
}



function obtenerEtiquetaPago(metodo) {
  return metodo || '—'
}

function formatearNumero(valor) {
  const numero = Number(valor) || 0
  return numero.toLocaleString('es-CO')
}

function formatearFecha(fechaISO) {
  if (!fechaISO) return 'Sin fecha'
  const fecha = new Date(fechaISO)
  if (isNaN(fecha.getTime())) return 'Sin fecha'
  return fecha.toLocaleString('es-CO', {
    day: '2-digit',
    month: 'short',
    year: 'numeric',
    hour: '2-digit',
    minute: '2-digit',
  })
}

function obtenerFechaHoraActual() {
  const ahora = new Date()
  const offset = ahora.getTimezoneOffset()
  const local = new Date(ahora.getTime() - offset * 60000)
  return local.toISOString().slice(0, 16)
}

let temporizadorAviso = null
function mostrarAviso(texto, tipo = 'exito') {
  mensajeAviso.value = texto
  mensajeAvisoTipo.value = tipo
  if (temporizadorAviso) clearTimeout(temporizadorAviso)
  temporizadorAviso = setTimeout(() => {
    mensajeAviso.value = ''
  }, 3000)
}
</script>

<style scoped>
.app-layout {
  background-color: #f1f5f9;
  color: #0f172a;
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
}

.header-taller {
  background: linear-gradient(135deg, #1e293b 0%, #0f172a 100%);
  border-bottom: 1px solid #334155;
}

.header-search-input {
  width: 380px;
  max-width: 100%;
  border-radius: 8px;
}

.btn-nuevo {
  border-radius: 8px;
  letter-spacing: 0.3px;
}

.page-content {
  background-color: #f1f5f9;
  min-height: 100vh;
}

/* Tarjetas de Estadísticas */
.stat-card {
  background-color: #ffffff;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  transition: all 0.2s ease-in-out;
}

.stat-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.06);
}

.stat-active {
  border: 2px solid #1976d2 !important;
  background-color: #eff6ff !important;
}

/* Barra de Control */
.control-bar {
  background-color: #ffffff;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
}

.border-top-subtle {
  border-top: 1px solid #f1f5f9;
}

/* Tarjetas de Dispositivos */
.device-card {
  background-color: #ffffff;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.04);
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.device-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.08);
}

.ribbon-card {
  letter-spacing: 0.5px;
  font-size: 12.5px;
}

/* Mini Stepper de Etapas */
.stepper-track {
  position: relative;
  margin: 4px 0;
  padding: 4px 2px;
}

.step-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
  z-index: 1;
  flex: 1;
  opacity: 0.45;
  transition: all 0.2s ease;
}

.step-item:hover {
  opacity: 0.85;
}

.step-dot {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background-color: #cbd5e1;
  color: #ffffff;
  font-size: 11.5px;
  font-weight: bold;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 2px;
}

.step-label {
  font-size: 11.5px;
  font-weight: 600;
  color: #64748b;
  white-space: nowrap;
}

.step-completed {
  opacity: 0.8;
}

.step-completed .step-dot {
  background-color: #10b981;
}

.step-completed .step-label {
  color: #059669;
}

.step-active {
  opacity: 1 !important;
  transform: scale(1.05);
}

.step-active .step-dot {
  background-color: #1976d2;
  box-shadow: 0 0 0 3px rgba(25, 118, 210, 0.25);
}

.step-active .step-label {
  color: #1976d2;
  font-weight: 700;
}

.obs-box {
  background-color: #f8fafc;
  border-left: 3px solid #cbd5e1;
  border-radius: 0 4px 4px 0;
}

.border-subtle {
  border: 1px solid #e2e8f0;
}



.empty-box {
  background-color: #ffffff;
  border: 2px dashed #cbd5e1;
  border-radius: 12px;
}

.modal-form-card {
  border-radius: 12px;
  overflow: hidden;
}

/* Toast Flotante */
.toast-notificacion {
  position: fixed;
  bottom: 24px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 9999;
  display: flex;
  align-items: center;
  padding: 12px 24px;
  border-radius: 50px;
  color: #ffffff;
  font-weight: 600;
  font-size: 14.5px;
  letter-spacing: 0.2px;
}

.toast-exito {
  background-color: #0f766e;
}

.toast-error {
  background-color: #b91c1c;
}

/* Transición Toast */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease, transform 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translate(-50%, 15px);
}


</style>