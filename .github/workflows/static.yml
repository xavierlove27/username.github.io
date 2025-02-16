<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>3D Online Department Store</title>
  <style>
    body { margin: 0; overflow: hidden; }
    canvas { display: block; }
    #infoPanel {
      position: absolute;
      top: 10px;
      left: 10px;
      background: rgba(255,255,255,0.95);
      padding: 15px;
      border: 1px solid #ccc;
      font-family: Arial, sans-serif;
      font-size: 14px;
      display: none;
      max-width: 300px;
    }
    #paymentModal {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      background: rgba(255,255,255,0.98);
      padding: 20px;
      border: 1px solid #ccc;
      font-family: Arial, sans-serif;
      font-size: 14px;
      display: none;
      z-index: 10;
      width: 300px;
    }
    #paymentModal input, #paymentModal button {
      width: 100%;
      margin: 5px 0;
      padding: 8px;
    }
    #overlay {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(0,0,0,0.5);
      display: none;
      z-index: 5;
    }
  </style>
</head>
<body>
  <!-- Product detail panel -->
  <div id="infoPanel"></div>
  <!-- Overlay for modal -->
  <div id="overlay"></div>
  <!-- Payment modal -->
  <div id="paymentModal">
    <h3>Payment Details</h3>
    <input type="text" id="cardNumber" placeholder="Card Number">
    <input type="text" id="expiry" placeholder="MM/YY">
    <input type="text" id="cvv" placeholder="CVV">
    <button id="submitPayment">Submit Payment</button>
    <div id="paymentStatus" style="margin-top:10px;"></div>
  </div>

  <!-- Three.js and OrbitControls from CDN -->
  <script src="https://cdn.jsdelivr.net/npm/three@0.152.0/build/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.152.0/examples/js/controls/OrbitControls.js"></script>
  <script>
    // Set up scene, camera, and renderer.
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
    camera.position.set(0, 10, 30);

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    document.body.appendChild(renderer.domElement);

    // OrbitControls allow you to move around.
    const controls = new THREE.OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;

    // Lighting.
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
    scene.add(ambientLight);
    const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
    directionalLight.position.set(10, 20, 10);
    scene.add(directionalLight);

    // Helper function: creates a room with a floor and four walls.
    // doorOptions (optional) adds a door opening on one wall.
    function createRoom(x, z, width, depth, doorOptions = null, roomName = "") {
      const room = new THREE.Group();

      // Floor.
      const floorGeo = new THREE.PlaneGeometry(width, depth);
      const floorMat = new THREE.MeshStandardMaterial({ color: 0xaaaaaa });
      const floor = new THREE.Mesh(floorGeo, floorMat);
      floor.rotation.x = -Math.PI/2;
      room.add(floor);

      const wallHeight = 8;
      const wallThickness = 0.5;
      const wallMat = new THREE.MeshStandardMaterial({ color: 0x888888 });
      const halfW = width / 2;
      const halfD = depth / 2;

      // North wall (assumed at z = -halfD). If doorOptions are provided and on the north wall, split the wall.
      if (doorOptions && doorOptions.wall === "north") {
        const doorWidth = doorOptions.width;
        const doorX = doorOptions.position; // door center relative to wall center.
        // Left segment.
        const leftWidth = (doorX - doorWidth/2) + halfW;
        if (leftWidth > 0) {
          const leftGeo = new THREE.BoxGeometry(leftWidth, wallHeight, wallThickness);
          const leftWall = new THREE.Mesh(leftGeo, wallMat);
          leftWall.position.set(-halfW + leftWidth/2, wallHeight/2, -halfD);
          room.add(leftWall);
        }
        // Right segment.
        const rightWidth = halfW - (doorX + doorWidth/2);
        if (rightWidth > 0) {
          const rightGeo = new THREE.BoxGeometry(rightWidth, wallHeight, wallThickness);
          const rightWall = new THREE.Mesh(rightGeo, wallMat);
          rightWall.position.set(doorX + doorWidth/2 + rightWidth/2, wallHeight/2, -halfD);
          room.add(rightWall);
        }
        // Top segment above door.
        const doorHeight = doorOptions.height || 4;
        if (doorHeight < wallHeight) {
          const topGeo = new THREE.BoxGeometry(doorWidth, wallHeight - doorHeight, wallThickness);
          const topWall = new THREE.Mesh(topGeo, wallMat);
          topWall.position.set(doorX, doorHeight + (wallHeight - doorHeight)/2, -halfD);
          room.add(topWall);
        }
      } else {
        // Full north wall.
        const northGeo = new THREE.BoxGeometry(width, wallHeight, wallThickness);
        const northWall = new THREE.Mesh(northGeo, wallMat);
        northWall.position.set(0, wallHeight/2, -halfD);
        room.add(northWall);
      }

      // South wall.
      const southGeo = new THREE.BoxGeometry(width, wallHeight, wallThickness);
      const southWall = new THREE.Mesh(southGeo, wallMat);
      southWall.position.set(0, wallHeight/2, halfD);
      room.add(southWall);

      // East wall.
      const eastGeo = new THREE.BoxGeometry(wallThickness, wallHeight, depth);
      const eastWall = new THREE.Mesh(eastGeo, wallMat);
      eastWall.position.set(halfW, wallHeight/2, 0);
      room.add(eastWall);

      // West wall.
      const westGeo = new THREE.BoxGeometry(wallThickness, wallHeight, depth);
      const westWall = new THREE.Mesh(westGeo, wallMat);
      westWall.position.set(-halfW, wallHeight/2, 0);
      room.add(westWall);

      room.position.set(x, 0, z);
      room.name = roomName;
      scene.add(room);
      return room;
    }

    // Create Main Room (30 x 30) with a door on its north wall.
    // Door centered at x=0, 6 units wide and 4 units high.
    const mainRoom = createRoom(0, 0, 30, 30, { wall: "north", width: 6, position: 0, height: 4 }, "Main Room");

    // Create Secondary Room (20 x 20) and attach it to Main Room’s north side.
    // Position it so its south wall aligns with the door opening.
    const secondaryRoom = createRoom(0, -30, 20, 20, null, "Secondary Room");
    // Adjust secondary room position so its south wall (half-depth = 10) meets main room’s north wall at z=-15.
    secondaryRoom.position.z = -25;

    // Product data with positions (world coordinates) and assigned rooms.
    const products = [];
    const productData = [
      // Products in Main Room.
      { id: 1, name: "Gadget Pro", price: "$199.99", position: { x: -8, y: 2, z: 5 } },
      { id: 2, name: "Smart Widget", price: "$99.99", position: { x: 0, y: 2, z: 0 } },
      { id: 3, name: "Home Helper", price: "$149.99", position: { x: 8, y: 2, z: -5 } },
      // Products in Secondary Room.
      { id: 4, name: "Fashion Fit", price: "$59.99", position: { x: -4, y: 2, z: -28 } },
      { id: 5, name: "Style Elite", price: "$89.99", position: { x: 4, y: 2, z: -22 } }
    ];

    // Create product boxes.
    const boxGeometry = new THREE.BoxGeometry(3, 3, 3);
    productData.forEach(prod => {
      const boxMaterial = new THREE.MeshStandardMaterial({ color: Math.random() * 0xffffff });
      const box = new THREE.Mesh(boxGeometry, boxMaterial);
      box.position.set(prod.position.x, prod.position.y, prod.position.z);
      box.userData = prod;
      scene.add(box);
      products.push(box);
    });

    // Set up raycaster for product selection.
    const raycaster = new THREE.Raycaster();
    const mouse = new THREE.Vector2();
    const infoPanel = document.getElementById('infoPanel');

    function onMouseClick(event) {
      mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
      mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
      raycaster.setFromCamera(mouse, camera);
      const intersects = raycaster.intersectObjects(products);
      if (intersects.length > 0) {
        const product = intersects[0].object.userData;
        showProductDetails(product);
      } else {
        infoPanel.style.display = "none";
      }
    }
    window.addEventListener('click', onMouseClick, false);

    // Display product details and a "Buy Now" button.
    function showProductDetails(product) {
      infoPanel.style.display = "block";
      infoPanel.innerHTML = `
        <strong>${product.name}</strong><br>
        Price: ${product.price}<br>
        <button id="buyNowButton">Buy Now</button>
      `;
      document.getElementById('buyNowButton').addEventListener('click', function(e) {
        e.stopPropagation();
        infoPanel.style.display = "none";
        showPaymentModal(product);
      });
    }

    // Payment modal functionality.
    const overlay = document.getElementById('overlay');
    const paymentModal = document.getElementById('paymentModal');
    const submitPayment = document.getElementById('submitPayment');
    const paymentStatus = document.getElementById('paymentStatus');

    function showPaymentModal(product) {
      overlay.style.display = "block";
      paymentModal.style.display = "block";
      paymentModal.setAttribute('data-product-id', product.id);
      paymentStatus.innerHTML = "";
    }

    submitPayment.addEventListener('click', function(e) {
      paymentStatus.innerHTML = "Processing payment...";
      submitPayment.disabled = true;
      // Simulate payment processing.
      setTimeout(() => {
        paymentStatus.innerHTML = "Payment successful!";
        submitPayment.disabled = false;
        setTimeout(closePaymentModal, 2000);
      }, 2000);
    });

    function closePaymentModal() {
      paymentModal.style.display = "none";
      overlay.style.display = "none";
      document.getElementById('cardNumber').value = "";
      document.getElementById('expiry').value = "";
      document.getElementById('cvv').value = "";
    }

    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });

    // Render loop.
    function animate() {
      requestAnimationFrame(animate);
      controls.update();
      renderer.render(scene, camera);
    }
    animate();
  </script>
</body>
</html>
