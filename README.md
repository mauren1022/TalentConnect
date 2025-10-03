# TalentConnect
Mauren Vega, Diego Guzman,Julian Amaya

from flask import Flask, request, render_template_string
import pika
import json
import uuid
from datetime import datetime

app = Flask(__name__)

def conectar_rabbitmq():
    try:
        connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
        channel = connection.channel()
        return connection, channel
    except Exception as e:
        return None, None

def enviar_mensaje(routing_key, mensaje):
    connection, channel = conectar_rabbitmq()
    if channel:
        try:
            channel.basic_publish(
                exchange='talentconnect',
                routing_key=routing_key,
                body=json.dumps(mensaje, ensure_ascii=False)
            )
            connection.close()
            return True
        except:
            return False
    return False

@app.route('/')
def inicio():
    return render_template_string('''
<!DOCTYPE html>
<html>
<head>
    <title>TalentConnect - Portal de Postulaciones</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 0; background: linear-gradient(135deg, #667eea, #764ba2); }
        .container { max-width: 900px; margin: 20px auto; background: white; border-radius: 15px; overflow: hidden; box-shadow: 0 10px 30px rgba(0,0,0,0.2); }
        .header { background: linear-gradient(135deg, #3498db, #2980b9); color: white; padding: 30px; text-align: center; }
        .header h1 { margin: 0; font-size: 2.5em; }
        .header p { margin: 10px 0 0 0; opacity: 0.9; }
        .section { padding: 30px; border-bottom: 1px solid #eee; }
        .section:last-child { border-bottom: none; }
        .section h2 { color: #2c3e50; margin-bottom: 20px; }
        .admin-section { background: linear-gradient(135deg, #e74c3c, #c0392b); color: white; }
        .admin-section h2 { color: white; }
        .form-group { margin-bottom: 20px; }
        .form-row { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
        label { display: block; margin-bottom: 5px; font-weight: bold; color: #34495e; }
        .admin-section label { color: white; }
        input, select, textarea { width: 100%; padding: 12px; border: 2px solid #bdc3c7; border-radius: 8px; font-size: 16px; box-sizing: border-box; }
        input:focus, select:focus, textarea:focus { border-color: #3498db; outline: none; }
        button { background: linear-gradient(135deg, #3498db, #2980b9); color: white; padding: 15px 30px; border: none; border-radius: 8px; font-size: 16px; cursor: pointer; transition: all 0.3s; }
        button:hover { transform: translateY(-2px); box-shadow: 0 5px 15px rgba(52,152,219,0.3); }
        .success { background: #d4edda; color: #155724; padding: 15px; border-radius: 8px; margin: 20px 0; border-left: 4px solid #28a745; }
        .status { background: #f8f9fa; padding: 20px; border-radius: 8px; text-align: center; }
        .status.connected { border-left: 4px solid #28a745; }
        .status.error { border-left: 4px solid #dc3545; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1> TalentConnect</h1>
            <p>Sistema Integral de Gestión de Postulaciones</p>
            <p><em>Únete a Nuestro Equipo - Powered by RabbitMQ</em></p>
        </div>

        <div class="section admin-section">
            <h2>🔧 Panel Administrativo</h2>
            <p style="margin-bottom: 20px;">Crear y gestionar vacantes laborales</p>
            <form action="/crear-vacante" method="post">
                <div class="form-row">
                    <div class="form-group">
                        <label>Título del Cargo *</label>
                        <input type="text" name="titulo" placeholder="Ej: Desarrollador Python Senior" required>
                    </div>
                    <div class="form-group">
                        <label>Departamento *</label>
                        <select name="departamento" required>
                            <option value="">Seleccionar departamento</option>
                            <option value="Tecnología">Tecnología</option>
                            <option value="Recursos Humanos">Recursos Humanos</option>
                            <option value="Ventas">Ventas</option>
                            <option value="Marketing">Marketing</option>
                            <option value="Operaciones">Operaciones</option>
                        </select>
                    </div>
                </div>
                <div class="form-row">
                    <div class="form-group">
                        <label>Rango Salarial</label>
                        <input type="text" name="salario" placeholder="Ej: $2.500.000 - $3.500.000">
                    </div>
                    <div class="form-group">
                        <label>Modalidad de Trabajo</label>
                        <select name="modalidad">
                            <option value="">Seleccionar modalidad</option>
                            <option value="Presencial">Presencial</option>
                            <option value="Remoto">Remoto</option>
                            <option value="Híbrido">Híbrido</option>
                        </select>
                    </div>
                </div>
                <div class="form-group">
                    <label>Requisitos del Cargo</label>
                    <textarea name="requisitos" rows="4" placeholder="Describe los requisitos, experiencia y habilidades necesarias..."></textarea>
                </div>
                <button type="submit">➕ Crear Vacante</button>
            </form>
        </div>

        <div class="section">
            <h2>👤 Portal de Postulaciones</h2>
            <p style="color: #7f8c8d; margin-bottom: 25px;">Envía tu hoja de vida y forma parte de nuestra familia TalentConnect</p>
            <form action="/postularse" method="post">
                <div class="form-group">
                    <label>Nombre Completo *</label>
                    <input type="text" name="nombre" placeholder="Ingresa tu nombre completo" required>
                </div>
                <div class="form-row">
                    <div class="form-group">
                        <label>Tipo de Documento *</label>
                        <select name="tipo_documento" required>
                            <option value="">Seleccione tipo</option>
                            <option value="CC">Cédula de Ciudadanía</option>
                            <option value="CE">Cédula de Extranjería</option>
                            <option value="PP">Pasaporte</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <label>Número de Documento *</label>
                        <input type="text" name="documento" placeholder="Número de documento" required>
                    </div>
                </div>
                <div class="form-row">
                    <div class="form-group">
                        <label>Correo Electrónico *</label>
                        <input type="email" name="email" placeholder="ejemplo@email.com" required>
                    </div>
                    <div class="form-group">
                        <label>Teléfono *</label>
                        <input type="tel" name="telefono" placeholder="300 123 4567" required>
                    </div>
                </div>
                <div class="form-row">
                    <div class="form-group">
                        <label>Cargo al que Aspira *</label>
                        <select name="cargo_aspirar" required>
                            <option value="">Seleccione un cargo</option>
                            <option value="Desarrollador Python">Desarrollador Python</option>
                            <option value="Analista de Datos">Analista de Datos</option>
                            <option value="Diseñador UX/UI">Diseñador UX/UI</option>
                            <option value="Gerente de Proyectos">Gerente de Proyectos</option>
                            <option value="Ingeniero de Software">Ingeniero de Software</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <label>Años de Experiencia</label>
                        <select name="experiencia">
                            <option value="">Seleccione</option>
                            <option value="Sin experiencia">Sin experiencia</option>
                            <option value="1-2 años">1-2 años</option>
                            <option value="3-5 años">3-5 años</option>
                            <option value="5+ años">5+ años</option>
                        </select>
                    </div>
                </div>
                <div class="form-group">
                    <label>Mensaje Personal (Opcional)</label>
                    <textarea name="mensaje" rows="3" placeholder="Cuéntanos por qué quieres formar parte de nuestro equipo..."></textarea>
                </div>
                <button type="submit">📄 Enviar Postulación</button>
            </form>
        </div>

        <div class="section">
            <h2>🔍 Consultar Estado de Postulación</h2>
            <form action="/consultar-estado" method="post">
                <div class="form-row">
                    <div class="form-group">
                        <label>Número de Documento</label>
                        <input type="text" name="documento" placeholder="Número de documento">
                    </div>
                    <div class="form-group">
                        <label>Correo Electrónico</label>
                        <input type="email" name="email" placeholder="ejemplo@email.com">
                    </div>
                </div>
                <button type="submit">🔍 Consultar Estado</button>
            </form>
        </div>
    </div>
</body>
</html>
    ''')

@app.route('/crear-vacante', methods=['POST'])
def crear_vacante():
    datos = request.form
    vacante_id = f"VAC-{uuid.uuid4().hex[:8].upper()}"
    
    mensaje_vacante = {
        "accion": "crear_vacante",
        "vacante_id": vacante_id,
        "titulo": datos.get('titulo'),
        "departamento": datos.get('departamento'),
        "requisitos": datos.get('requisitos'),
        "salario": datos.get('salario'),
        "modalidad": datos.get('modalidad'),
        "timestamp": datetime.now().isoformat()
    }
    
    if enviar_mensaje('admin.vacante.crear', mensaje_vacante):
        return f'''
        <div style="text-align: center; padding: 50px; font-family: Arial;">
            <div style="background: #d4edda; color: #155724; padding: 30px; border-radius: 10px; margin-bottom: 20px;">
                <h2> Vacante creada exitosamente!</h2>
                <p><strong>ID:</strong> {vacante_id}</p>
                <p><strong>Título:</strong> {datos.get('titulo')}</p>
                <p><strong>Departamento:</strong> {datos.get('departamento')}</p>
                <p> Mensaje enviado a RabbitMQ</p>
            </div>
            <a href="/" style="background: #3498db; color: white; padding: 15px 30px; text-decoration: none; border-radius: 8px;">🏠 Volver al Inicio</a>
        </div>
        '''
    else:
        return '<h2> Error creando vacante</h2><a href="/">Volver</a>'

@app.route('/postularse', methods=['POST'])
def postularse():
    datos = request.form
    postulacion_id = f"POST-{uuid.uuid4().hex[:8].upper()}"
    
    mensaje_postulacion = {
        "accion": "nueva_postulacion",
        "postulacion_id": postulacion_id,
        "candidato": {
            "nombre": datos.get('nombre'),
            "tipo_documento": datos.get('tipo_documento'),
            "documento": datos.get('documento'),
            "email": datos.get('email'),
            "telefono": datos.get('telefono')
        },
        "cargo_aspirar": datos.get('cargo_aspirar'),
        "experiencia": datos.get('experiencia'),
        "mensaje": datos.get('mensaje'),
        "timestamp": datetime.now().isoformat()
    }
    
    if enviar_mensaje('usuario.postulacion.nueva', mensaje_postulacion):
        return f'''
        <div style="text-align: center; padding: 50px; font-family: Arial;">
            <div style="background: #d4edda; color: #155724; padding: 30px; border-radius: 10px; margin-bottom: 20px;">
                <h2> Postulación enviada exitosamente!</h2>
                <p><strong>ID:</strong> {postulacion_id}</p>
                <p><strong>Candidato:</strong> {datos.get('nombre')}</p>
                <p><strong>Cargo:</strong> {datos.get('cargo_aspirar')}</p>
                <p> Recibirás confirmación por email</p>
                <p> Mensaje enviado a RabbitMQ</p>
            </div>
            <a href="/" style="background: #3498db; color: white; padding: 15px 30px; text-decoration: none; border-radius: 8px;">🏠 Volver al Inicio</a>
        </div>
        '''
    else:
        return '<h2> Error enviando postulación</h2><a href="/">Volver</a>'

@app.route('/consultar-estado', methods=['POST'])
def consultar_estado():
    datos = request.form
    
    mensaje_consulta = {
        "accion": "consultar_estado",
        "documento": datos.get('documento'),
        "email": datos.get('email'),
        "timestamp": datetime.now().isoformat()
    }
    
    if enviar_mensaje('usuario.estado.consultar', mensaje_consulta):
        return f'''
        <div style="text-align: center; padding: 50px; font-family: Arial;">
            <div style="background: #e3f2fd; color: #1565c0; padding: 30px; border-radius: 10px; margin-bottom: 20px;">
                <h2> Estado de tu Postulación</h2>
                <p><strong>Estado Actual:</strong> 👁️ En Revisión</p>
                <p><strong>Última Actualización:</strong> {datetime.now().strftime('%Y-%m-%d %H:%M')}</p>
                <p> Te contactaremos pronto con novedades</p>
                <p> Consulta procesada via RabbitMQ</p>
            </div>
            <a href="/" style="background: #3498db; color: white; padding: 15px 30px; text-decoration: none; border-radius: 8px;">🏠 Volver al Inicio</a>
        </div>
        '''
    else:
        return '<h2> Error procesando consulta</h2><a href="/">Volver</a>'

if __name__ == '__main__':
    print(" Iniciando TalentConnect...")
    connection, channel = conectar_rabbitmq()
    if connection:
        print(" RabbitMQ conectado")
        connection.close()
        print(" Aplicación disponible en: http://localhost:5001")
    else:
        print(" RabbitMQ no disponible")
    
    app.run(debug=True, host='0.0.0.0', port=5001)


