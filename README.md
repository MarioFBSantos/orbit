# orbit3d (orbit3d)

- Esse projeto tem intuito de criar um exemplo de Three.Js em Vue 3 Script Setup (Composition API), já que não há exemplos na internet, então adaptei o modelo para tornar um modelo público para utilização. Além de ser um modelo, serve também como uma representação da órbita da terra em relação ao sol e da lua em relação a terra, na velocidade real. 

- Qualquer bug encontrado, gerar um Issue https://github.com/MarioFBSantos/orbit/issues

- Toda a lógica e funções você encontrará em https://github.com/MarioFBSantos/orbit/blob/master/src/pages/IndexPage.vue

# Escala

- A escala deste projeto será de 1/10000, neste momento a escala ainda não está assertiva.

# Iluminação

- A iluminação dos corpos selestes, está sendo feita pelo sol, existe uma fonte de emissão de luz dentro do modelo 3d do sol

# Orbitas

- A orbita da terra em relação ao sol está sendo definida nesta parte.
- - Foi utilizada a biblioteca SunCalc para auxiliar na posição da terra e calculo da velocidade.
- - Da para verificar a posição e velocidade da terra pelo console do navegador

- <code>   const moonPhi = ((90 - (moonPos.altitude * 180) / Math.PI) * Math.PI) / 180;
  const moonTheta = (((moonPos.azimuth * 180) / Math.PI - 90) * Math.PI) / 180;
  const moonX = moonDist * Math.sin(moonPhi) * Math.cos(moonTheta);
  const moonY = moonDist * Math.sin(moonPhi) * Math.sin(moonTheta);
  const moonZ = moonDist * Math.cos(moonPhi);
 </code> A orbita da lua em relação a terra
 
 - <code>   const pos = SunCalc.getPosition(date, 0, 0);
  const dist = 150; // Distância média da Terra ao Sol em milhões de km
  const phi = ((90 - (pos.altitude * 180) / Math.PI) * Math.PI) / 180;
  const theta = (((pos.azimuth * 180) / Math.PI - 90) * Math.PI) / 180;
  const x = dist * Math.sin(phi) * Math.cos(theta);
  const y = dist * Math.sin(phi) * Math.sin(theta);
  const z = dist * Math.cos(phi);
  </code> Posição heliocêntrica da terra


# Posições

-  <code>moon.position.set(130, 0, 0);</code> Ajusta a posição da Lua em relação à Terra (não a distância, a distância é dentro da função 'animate()' onde tem a variável <code>moonDistance = 30;</code>
- <code>  earth.position.setFromSphericalCoords(
    distance,
    azimuth - Math.PI / 2,
    altitude
  );</code> Atualização da posição da terra em relação ao sol
- <code>earth.position.set(150, 0, 0);</code> Ajusta a posição da Terr em relação ao Sol

# Câmera

- <code>camera.position.set(50, 50, 50);</code> Posição da câmera
- <code>controls.maxDistance = 10000;</code> Aumenta distância máxima da câmera
- <code>controls.rotateSpeed = 0.5;</code> Velocidade de rotação da câmera

# Futuro

- Implementar todos os planetas
- Câmera livre
- Velocidade real de todos os astros
- Movimentos de translação
- Componentizar funções
- Comportamento gravitacional que um corpo celeste exerce pelo o outro

# Licenças

- https://github.com/MarioFBSantos/orbit/tree/master/public/images Em cada pasta, contém a licença de cada asset em 3D, não modelei nenhum asset e não pretendo utilizar os modelos 3d para intuitos financeiros.
