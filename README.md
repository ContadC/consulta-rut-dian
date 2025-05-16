const express = require('express');
const puppeteer = require('puppeteer');

const app = express();

app.get('/rut', async (req, res) => {
  const nit = req.query.nit;
  if (!nit) return res.status(400).json({ error: 'NIT requerido' });

  const browser = await puppeteer.launch({
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  });
  const page = await browser.newPage();

  try {
    await page.goto('https://muisca.dian.gov.co/WebRut/ConsultaEstadoRUT.faces');

    await page.type('input[name="formConsultaEstadoRUT:inputNIT"]', nit);
    await page.click('input[name="formConsultaEstadoRUT:btnBuscar"]');
    await page.waitForSelector('#formConsultaEstadoRUT\\:j_idt51');

    const resultado = await page.evaluate(() => {
      const datos = document.querySelector('#formConsultaEstadoRUT\\:j_idt51')?.innerText || '';
      return { resultado: datos };
    });

    await browser.close();
    res.json({ nit, ...resultado });

  } catch (err) {
    await browser.close();
    res.status(500).json({ error: 'Error en la consulta', details: err.message });
  }
});

app.listen(3000, () => {
  console.log('Servicio de consulta RUT corriendo en puerto 3000');
});
