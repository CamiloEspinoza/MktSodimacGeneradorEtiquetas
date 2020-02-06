<template>
  <div id="app">
    <h1>Generador Etiquetas MKT Sodimac</h1>

    <textarea v-model="datos" rows="8"></textarea>
    <br /><br />

    <div class="datos_descifrados" v-if="tabla">
      <h2>Datos Descifrados</h2>

      <table>
        <tr>
          <th>ASN</th>
          <th>Fecha Retiro</th>
          <th>Nº OC</th>
          <th>Fecha OC</th>
          <th>Etiqueta</th>
          <th>Cantidad</th>
          <th>Producto</th>
          <th>Precio Neto</th>
          <th>Guía</th>
        </tr>
        <tr v-for="fila in tabla" :key="fila.etiqueta">
          <td>{{ fila.asn }}</td>
          <td>{{ fila.fecha_retiro }}</td>
          <td>{{ fila.numero_oc }}</td>
          <td>{{ fila.fecha_oc }}</td>
          <td>{{ fila.etiqueta }}</td>
          <td>{{ fila.cantidad }}</td>
          <td>{{ fila.producto }}</td>
          <td>{{ fila.precio_neto }}</td>
          <td>{{ fila.guia }}</td>
        </tr>
      </table>

      <br /><br />
    </div>

    <button type="button" name="button" @click="generarEtiquetas">
      Generar Etiquetas
    </button>
    <div>
      <img id="itf" />
    </div>
  </div>
</template>

<script>
import JsBarcode from "jsbarcode";
import * as jsPDF from "jspdf";
// import canvg from "canvg";
export default {
  name: "app",
  components: {},
  data() {
    return {
      datos: "",
      productos: {
        Pedestal: {
          nombre: 'MKP VENTILADOR METAL PED 16"',
          codigo: "2000004018238"
        },
        Piso: {
          nombre: 'MKP VENTILADOR METAL PISO 16"',
          codigo: "2000004018252"
        },
        Mesa: {
          nombre: 'MKP VENTILADOR METAL MESA 12"',
          codigo: "2000004018245"
        }
      }
    };
  },
  mounted() {},
  methods: {
    generarEtiquetas() {
      const img = document.querySelector("img#itf");

      var doc = new jsPDF({
        orientation: "l",
        unit: "mm",
        format: [141.68, 283.6]
      });

      doc.setFont("courier", "normal");

      for (let [index, fila] of this.tabla.entries()) {
        for (let i = 0; i < fila.cantidad; i++) {
          JsBarcode("#itf", fila.etiqueta, {
            margin: 0,
            displayValue: false
          });

          doc.setFontSize(16);
          doc.text("CD: 214", 4, 8);
          doc.text("RSV: " + fila.asn, 46, 8);
          doc.addImage(img.src, "JPEG", 4, 10, 92, 30);
          doc.text(fila.etiqueta, 20, 45);

          //Etiqueta Producto
          doc.addPage([141.68, 283.6], "l");
          doc.setFontSize(14);
          doc.text(this.productos[fila.producto].nombre, 10, 8);
          JsBarcode("#itf", this.productos[fila.producto].codigo, {
            margin: 0,
            displayValue: true,
            format: "EAN13"
          });
          doc.addImage(img.src, "JPEG", 7, 12, 80, 34);

          if (index < this.tabla.length - 1 || i < fila.cantidad - 1) {
            doc.addPage([141.68, 283.6], "l");
          }
        }
      }

      doc.save(this.tabla[0].asn + ".pdf");
    }
  },
  computed: {
    tabla() {
      if (this.datos) {
        let tabla = [];
        let filas = this.datos.split("\n");

        for (let index in filas) {
          let fila = filas[index].split("\t");
          tabla.push({
            asn: fila[0],
            fecha_retiro: fila[1],
            numero_oc: fila[2],
            fecha_oc: fila[3],
            etiqueta: fila[4],
            cantidad: fila[5],
            producto: fila[6],
            precio_neto: fila[7],
            guia: fila[8]
          });
        }
        return tabla;
      } else {
        return null;
      }
    }
  }
};
</script>

<style lang="scss">
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

#itf {
  display: none;
}

textarea {
  width: 95%;
}

table {
  width: 100%;
  border-collapse: collapse;

  td {
    border: 1px solid #ccc;
  }
}
</style>
