<script>
  import { onMount } from 'svelte';
  import Globe from 'globe.gl';
  import * as d3 from 'd3-geo';
  import { feature } from 'topojson-client';
  import * as THREE from 'three';
  import { countryCapitals } from './countries.js';

  let globeEl;
  let landPolygons;
  let animationFrame;
  let pulseValue = 0;

  async function loadLandPolygons() {
    const res = await fetch('https://unpkg.com/world-atlas@2.0.2/countries-50m.json');
    const worldData = await res.json();
    landPolygons = feature(worldData, worldData.objects.countries);
  }

  function isOnLand(lat, lng) {
    return landPolygons && d3.geoContains(landPolygons, [lng, lat]);
  }

  function generateEvenlySpacedLandPoints(n) {
    const points = [];
    let attempts = 0;
    while (points.length < n && attempts < n * 20) {
      const z = 2 * Math.random() - 1;
      const theta = 2 * Math.PI * Math.random();
      const lat = Math.asin(z) * (180 / Math.PI);
      const lng = theta * (180 / Math.PI) - 180;
      if (isOnLand(lat, lng)) {
        points.push({ lat, lng });
      }
      attempts++;
    }
    return points;
  }

  onMount(async () => {
    await loadLandPolygons();

    const globePoints = generateEvenlySpacedLandPoints(countryCapitals.length);
    const animatedPoints = countryCapitals.slice(0, globePoints.length).map((point, idx) => ({
      ...point,
      lat: globePoints[idx].lat,
      lng: globePoints[idx].lng
    }));

    const globe = new Globe()
      .globeImageUrl('https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg')
      .bumpImageUrl('https://unpkg.com/three-globe/example/img/earth-topology.png')
      .backgroundColor('#ffffff')
      .pointsData(animatedPoints)
      .pointsMerge(false)
      .pointLat('lat')
      .pointLng('lng')
      .pointAltitude(0)
      .pointLabel('capital')
      .labelSize(0.5)
      .labelDotRadius(0.5)
      .labelColor(() => '#ffec8b')
      .labelAltitude(0.01)
      .onPointClick((point) => {
        alert(`Capital: ${point.capital}\nCountry: ${point.country}`);
      })
      .pointColor((d) =>
        d.parentPoint ? 'rgba(246, 139, 31, 0.4)' : '#F68B1F'
      )
      .customThreeObject((d) => {
        const group = new THREE.Group();

        // Larger orange core with glow effect
        const coreMaterial = new THREE.MeshStandardMaterial({
          color: '#F68B1F',
          emissive: '#F68B1F',
          emissiveIntensity: 1.5
        });
        const coreGeometry = new THREE.SphereGeometry(0.12, 16, 16);
        const coreMesh = new THREE.Mesh(coreGeometry, coreMaterial);
        group.add(coreMesh);

        // Larger gold pulsing ring
        const ringMaterial = new THREE.MeshBasicMaterial({
          color: '#FFD700',
          transparent: true,
          opacity: 0.7,
          side: THREE.DoubleSide,
          depthWrite: false
        });
        const ringGeometry = new THREE.RingGeometry(0.13, 0.16, 64);
        const ringMesh = new THREE.Mesh(ringGeometry, ringMaterial);
        ringMesh.rotation.x = -Math.PI / 2;
        group.add(ringMesh);

        d.__ringMesh = ringMesh;
        d.__coreMesh = coreMesh;
        return group;
      })(globeEl);

    const ambientLight = new THREE.AmbientLight(0xffffff, 1.5);
    globe.scene().add(ambientLight);

    globe.controls().autoRotate = true;
    globe.controls().autoRotateSpeed = 1.0;
    globe.pointOfView({ lat: 0, lng: 0, altitude: 2.5 }, 0);

    const animate = () => {
      pulseValue += 0.015;
      const ringScale = 1 + 1.5 * Math.abs(Math.sin(pulseValue));
      const ringOpacity = 0.7 * (1 - Math.abs(Math.sin(pulseValue)));
      const coreScale = 1 + 0.3 * Math.sin(pulseValue * 1.2);

      globe.pointsData().forEach((d) => {
        if (d.__ringMesh) {
          d.__ringMesh.scale.set(ringScale, ringScale, ringScale);
          d.__ringMesh.material.opacity = ringOpacity;
        }
        if (d.__coreMesh) {
          d.__coreMesh.scale.set(coreScale, coreScale, coreScale);
        }
      });

      animationFrame = requestAnimationFrame(animate);
    };

    animate();

    return () => cancelAnimationFrame(animationFrame);
  });
</script>

<style>
	 @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');

  :global(body) {
    font-family: 'Inter', sans-serif;
  }
  .globe-container {
    width: 100%;
    height: 100%;
    background-color: white;
		position: relative;
		display: flex;
		overflow: hidden;
		justify-content: flex-end;
    align-items: center;
  }
	 .logo {
    position: absolute;
    top: 20px;
    left: 20px;
    width: 180px;
    z-index: 10;
  }
.globe-wrapper {
  position: relative;
  width: 1000px;
  height: 1000px;
  transform: scale(0.7) translateY(400px) translateX(0px);
  transform-origin: top right;
  margin-left: auto;
	margin-right: -110px;
}
	.section {
    display: flex;
    justify-content: space-between;
    align-items: center;
    height: 680px;
    padding: 80px 60px;
    background-color: white;
    box-sizing: border-box;
    overflow: hidden;
}
.text-content {
  width: 100%;
  max-width: 1000px;
  padding-right: 20px;
}

.cta-button:hover {
  background-color: #d57717;
}
.cta-button {
  background-color: #F68B1F;
  color: white;
  padding: 12px 24px;
  border: none;
  border-radius: 6px;
  font-size: 1rem;
  cursor: pointer;
  transition: background-color 0.3s ease;
}
.subheadline {
  font-size: 1.625rem;
  font-weight: 400;
  color: #374151;;
  margin-bottom: 24px;
  line-height: 1.5;
}
.divider {
  border: none;
  height: 1px;
  background-color: #e0e0e0;
  width: 100%;
  margin: 0 0 24px 0;
}
.download-stat {
  display: flex;
  align-items: baseline;
  gap: 10px;
  margin-bottom: 30px;
}
.download-number {
  font-size: 2rem;
  font-weight: 700;
  color: #F68B1F;
}
.download-label {
  font-size: 1.125rem;
  color: #4A4A4A;
  margin-top: 8px;
  font-weight: 400;
}
	.download-text {
  display: flex;
  flex-direction: column;
}
	
</style>
<div class="section">
  <div class="text-content">
    <p class="subheadline">Connect with your customers anywhere in the world using our open-source help desk solution.</p>

		<hr class="divider" />

		
		<div class="download-stat"> 
<div class="download-text">
    <div class="download-number">5 million+</div>
    <div class="download-label">downloads and counting</div>
  </div>
	  </div>
		 <button class="cta-button">Learn More </button>
		  </div>

	
	
<div class="globe-container">
  <div class="globe-wrapper" bind:this={globeEl}></div>
</div>
</div>
 <a href="https://ibb.co/BHrRw81h">
    <img src="https://i.ibb.co/j9WFH7pC/Screenshot-2025-07-02-at-11-44-33-AM.png" alt="osTicket Logo" class="logo" />
  </a>
