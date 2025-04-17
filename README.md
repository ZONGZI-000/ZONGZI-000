## Hi there 👋

<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>公众号关注：前端Hardy</title>
    <script src="https://cdn.jsdelivr.net/npm/three/build/three.min.js"></script>
    <style>
        html,
        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
        }
    </style>
</head>

<body>

    <head>
        <title>Interaction</title>
    </head>
    <script>
        // 图标地址太长~后台联系我私发
        let texture = ''
        let shadow_texture = ''

        var scene = new THREE.Scene();

        let aspect = window.innerWidth / window.innerHeight;
        let camera_distance = 8;
        const camera = new THREE.OrthographicCamera(
            -camera_distance * aspect,
            camera_distance * aspect,
            camera_distance,
            -camera_distance,
            0.01,
            1000
        );

        camera.position.set(-0, -10, 5);
        camera.lookAt(0, 0, 0);

        var renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);
        scene.background = new THREE.Color(0xffffff);

        const geometry_sphere = new THREE.SphereGeometry(0.25, 32, 16);
        const material_sphere = new THREE.MeshBasicMaterial({
            color: 0xff0000,
            transparent: true,
            opacity: 0,
            depthWrite: false
        });
        const sphere = new THREE.Mesh(geometry_sphere, material_sphere);
        scene.add(sphere);

        var geometry_hit = new THREE.PlaneGeometry(500, 500, 10, 10);
        const material_hit = new THREE.MeshBasicMaterial({
            color: 0xffffff,
            transparent: true,
            opacity: 0,
            depthWrite: false
        });
        var hit = new THREE.Mesh(geometry_hit, material_hit);
        hit.name = "hit";
        scene.add(hit);

        var geometry = new THREE.PlaneGeometry(15, 15, 100, 100);
        let shader_material = new THREE.ShaderMaterial({
            uniforms: {
                uTexture: { type: "t", value: new THREE.TextureLoader().load(texture) },
                uDisplacement: { value: new THREE.Vector3(0, 0, 0) }
            },

            vertexShader: `
  varying vec2 vUv;
  uniform vec3 uDisplacement;

float easeInOutCubic(float x) {
  return x < 0.5 ? 4. * x * x * x : 1. - pow(-2. * x + 2., 3.) / 2.;
}

float map(float value, float min1, float max1, float min2, float max2) {
  return min2 + (value - min1) * (max2 - min2) / (max1 - min1);
}  

  void main() {
   vUv = uv;
   vec3 new_position = position; 

    vec4 localPosition = vec4( position, 1.);
    vec4 worldPosition = modelMatrix * localPosition;

    float dist = (length(uDisplacement - worldPosition.rgb));
    float min_distance = 3.;

    if (dist < min_distance){
      float distance_mapped = map(dist, 0., min_distance, 1., 0.);
      float val = easeInOutCubic(distance_mapped) * 1.; 
      new_position.z +=  val;
    }
    
   
   gl_Position = projectionMatrix * modelViewMatrix * vec4(new_position,1.0);
  }
`,
            fragmentShader: ` 
    varying vec2 vUv;
    uniform sampler2D uTexture;
    void main()
    {
       vec4 color =  texture2D(uTexture, vUv); 
       gl_FragColor = vec4(color) ;    
    }`,
            transparent: true,
            depthWrite: false,
            side: THREE.DoubleSide
        });

        var plane = new THREE.Mesh(geometry, shader_material);
        plane.rotation.z = Math.PI / 4;
        scene.add(plane);

        let shader_material_shadow = new THREE.ShaderMaterial({
            uniforms: {
                uTexture: {
                    type: "t",
                    value: new THREE.TextureLoader().load(shadow_texture)
                },
                uDisplacement: { value: new THREE.Vector3(0, 0, 0) }
            },

            vertexShader: `
  varying vec2 vUv;
  varying float dist;
  uniform vec3 uDisplacement;

  void main() {
   vUv = uv;
   
   vec4 localPosition = vec4( position, 1.);
   vec4 worldPosition = modelMatrix * localPosition;
   dist = (length(uDisplacement - worldPosition.rgb));
   
   gl_Position = projectionMatrix * modelViewMatrix * vec4(position,1.0);
  }
`,
            fragmentShader: ` 
    varying vec2 vUv;
    varying float dist;
    uniform sampler2D uTexture;
    
    float map(float value, float min1, float max1, float min2, float max2) {
  return min2 + (value - min1) * (max2 - min2) / (max1 - min1);
}  

    void main()
    {
       vec4 color =  texture2D(uTexture, vUv); 
       float min_distance = 3.;

       if (dist < min_distance){
        float alpha = map(dist, min_distance, 0., color.a , 0.);
        color.a  = alpha;
        }
       
       gl_FragColor = vec4(color) ;    
    }`,
            transparent: true,
            depthWrite: false,
            side: THREE.DoubleSide
        });

        var plane_shadow = new THREE.Mesh(geometry, shader_material_shadow);
        plane_shadow.rotation.z = Math.PI / 4;
        scene.add(plane_shadow);

        window.addEventListener("resize", onWindowResize);

        const raycaster = new THREE.Raycaster();
        const pointer = new THREE.Vector2();
        window.addEventListener("pointermove", onPointerMove);

        function onPointerMove(event) {
            pointer.x = (event.clientX / window.innerWidth) * 2 - 1;
            pointer.y = -(event.clientY / window.innerHeight) * 2 + 1;

            raycaster.setFromCamera(pointer, camera);
            const intersects = raycaster.intersectObject(hit);

            if (intersects.length > 0) {
                sphere.position.set(
                    intersects[0].point.x,
                    intersects[0].point.y,
                    intersects[0].point.z
                );

                shader_material.uniforms.uDisplacement.value = sphere.position;
                shader_material_shadow.uniforms.uDisplacement.value = sphere.position;
            }
        }

        function render() {
            requestAnimationFrame(render);
            renderer.render(scene, camera);
        }

        render();

        function onWindowResize() {
            aspect = window.innerWidth / window.innerHeight;

            camera.left = -camera_distance * aspect;
            camera.right = camera_distance * aspect;
            camera.top = camera_distance;
            camera.bottom = -camera_distance;

            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }
    </script>
</body>

</html>




<!--
**ZONGZI-000/ZONGZI-000** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...
-->
